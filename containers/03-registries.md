# Container Registries

> **What it is:** Container registries are repositories for storing and distributing container images. They can be public (Docker Hub, Quay.io) or private (self-hosted, cloud-based).

## Registry Types

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Container Registries                                 │
│                                                                              │
│  PUBLIC                    PRIVATE (Cloud)           PRIVATE (Self-hosted)  │
│  ┌────────────────┐       ┌────────────────┐        ┌────────────────┐     │
│  │ Docker Hub     │       │ AWS ECR        │        │ Harbor         │     │
│  │ quay.io        │       │ GCP GCR/AR     │        │ GitLab Reg     │     │
│  │ ghcr.io        │       │ Azure ACR      │        │ Nexus          │     │
│  │ gcr.io         │       │ Red Hat Quay   │        │ JFrog          │     │
│  └────────────────┘       └────────────────┘        └────────────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Image Naming Convention

```bash
# Full image reference
[registry/][namespace/]repository[:tag][@digest]

# Examples
nginx                                    # Docker Hub official: docker.io/library/nginx:latest
nginx:1.25                              # With tag
myuser/myapp                            # Docker Hub user: docker.io/myuser/myapp:latest
quay.io/myorg/myapp:v1.0                # Quay.io
ghcr.io/myuser/myapp:latest             # GitHub Container Registry
registry.example.com/myapp:1.0          # Private registry
gcr.io/project/myapp@sha256:abc123...   # With digest (immutable)
```

---

## Quay.io (Red Hat)

> **What it is:** Quay.io is Red Hat's enterprise container registry with security scanning, access controls, geo-replication, and integration with OpenShift.

### Registration & Setup

```bash
# 1. Create account at https://quay.io
# 2. Create an organization or use personal namespace
# 3. Generate robot account for CI/CD (Settings > Robot Accounts)
```

### Authentication Methods

```bash
# Method 1: Interactive login
docker login quay.io
podman login quay.io

# Method 2: Using credentials directly
docker login -u="myorg+myrobot" -p="TOKEN" quay.io

# Method 3: Using auth file (for automation)
cat > ~/.docker/config.json << EOF
{
  "auths": {
    "quay.io": {
      "auth": "$(echo -n 'username:password' | base64)"
    }
  }
}
EOF

# Method 4: Podman auth file
podman login quay.io --authfile ~/.config/containers/auth.json
```

### Push to Quay.io

```bash
# Step 1: Build your image
docker build -t myapp:1.0 .

# Step 2: Tag for Quay.io
# Format: quay.io/<organization>/<repository>:<tag>
docker tag myapp:1.0 quay.io/myorg/myapp:1.0
docker tag myapp:1.0 quay.io/myorg/myapp:latest

# Step 3: Push
docker push quay.io/myorg/myapp:1.0
docker push quay.io/myorg/myapp:latest

# Push all tags at once
docker push -a quay.io/myorg/myapp

# Using podman
podman push quay.io/myorg/myapp:1.0
```

### Pull from Quay.io

```bash
# Pull public image
docker pull quay.io/myorg/myapp:1.0

# Pull private image (must be logged in)
docker login quay.io
docker pull quay.io/myorg/private-app:1.0

# Pull specific digest (immutable)
docker pull quay.io/myorg/myapp@sha256:abc123...
```

### Using Robot Accounts for CI/CD

```bash
# 1. Create robot account in Quay.io UI:
#    Organization > Robot Accounts > Create Robot Account

# 2. Grant permissions to repositories

# 3. Use in CI/CD pipeline (GitHub Actions example)
# - name: Login to Quay.io
#   run: docker login -u="${{ secrets.QUAY_USER }}" -p="${{ secrets.QUAY_TOKEN }}" quay.io

# 4. Kubernetes pull secret
kubectl create secret docker-registry quay-secret \
  --docker-server=quay.io \
  --docker-username="myorg+robot" \
  --docker-password="TOKEN" \
  --docker-email="email@example.com"
```

### Quay.io with skopeo

```bash
# Copy image to Quay.io
skopeo copy \
  docker://docker.io/library/nginx:latest \
  docker://quay.io/myorg/nginx:latest

# Copy between registries
skopeo copy \
  docker://registry.redhat.io/ubi9/ubi-minimal:latest \
  docker://quay.io/myorg/ubi-minimal:latest

# Inspect image without pulling
skopeo inspect docker://quay.io/myorg/myapp:1.0

# Sync entire repository
skopeo sync --src docker --dest docker \
  quay.io/myorg/myapp \
  backup-registry.example.com/myorg
```

### OpenShift with Quay.io

```yaml
# Create pull secret for Quay.io in OpenShift
oc create secret docker-registry quay-pull-secret \
  --docker-server=quay.io \
  --docker-username="myorg+robot" \
  --docker-password="TOKEN"

# Link to service account
oc secrets link default quay-pull-secret --for=pull

# Use in Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: myapp
        image: quay.io/myorg/myapp:1.0
      imagePullSecrets:
      - name: quay-pull-secret
```

---

## Docker Hub

```bash
# Login
docker login
docker login -u username -p password

# Push to Docker Hub
docker tag myapp:1.0 myuser/myapp:1.0
docker push myuser/myapp:1.0

# Pull from Docker Hub
docker pull nginx:latest
docker pull myuser/myapp:1.0
```

---

## Red Hat Registries

```bash
# registry.redhat.io - Requires Red Hat subscription
docker login registry.redhat.io
docker pull registry.redhat.io/ubi9/ubi-minimal
docker pull registry.redhat.io/rhel9/postgresql-15

# Common Red Hat base images
registry.redhat.io/ubi9/ubi                    # Full UBI
registry.redhat.io/ubi9/ubi-minimal            # Minimal (microdnf)
registry.redhat.io/ubi9/ubi-micro              # Micro (no package manager)
registry.redhat.io/ubi9/ubi-init               # Systemd support
```

---

## GitHub Container Registry (ghcr.io)

```bash
# Login with PAT (Personal Access Token)
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Push
docker tag myapp:1.0 ghcr.io/username/myapp:1.0
docker push ghcr.io/username/myapp:1.0

# Pull (public images don't need auth)
docker pull ghcr.io/username/myapp:1.0
```

---

## AWS ECR

```bash
# Get login token
aws ecr get-login-password --region us-east-1 | \
    docker login --username AWS --password-stdin \
    123456789.dkr.ecr.us-east-1.amazonaws.com

# Create repository
aws ecr create-repository --repository-name myapp

# Push
docker tag myapp:1.0 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0
```

---

## Private Registry Setup

```bash
# Run local registry
docker run -d -p 5000:5000 --name registry registry:2

# Push to local registry
docker tag myapp:1.0 localhost:5000/myapp:1.0
docker push localhost:5000/myapp:1.0

# With authentication
htpasswd -Bc /auth/htpasswd myuser

docker run -d -p 5000:5000 \
    -v /auth:/auth \
    -e REGISTRY_AUTH=htpasswd \
    -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
    -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
    registry:2
```

---

*Part of the [Containers Documentation](README.md)*

