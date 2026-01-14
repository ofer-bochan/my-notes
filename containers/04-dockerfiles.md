# Example Dockerfiles for Common Services

> **What it is:** This guide provides production-ready Dockerfile examples for common services including web servers, databases, and monitoring tools.

## Nginx Web Server

```dockerfile
# ============================================
# Nginx - Static Content Server
# ============================================
FROM nginx:1.25-alpine

LABEL maintainer="your-email@example.com"
LABEL description="Nginx web server for static content"

# Remove default config
RUN rm /etc/nginx/conf.d/default.conf

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf
COPY conf.d/ /etc/nginx/conf.d/

# Copy static website files
COPY html/ /usr/share/nginx/html/

# Create non-root user
RUN addgroup -g 1001 -S nginx-app && \
    adduser -u 1001 -S nginx-app -G nginx-app && \
    chown -R nginx-app:nginx-app /usr/share/nginx/html && \
    chown -R nginx-app:nginx-app /var/cache/nginx && \
    touch /var/run/nginx.pid && \
    chown nginx-app:nginx-app /var/run/nginx.pid

EXPOSE 80 443

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

**Example nginx.conf:**
```nginx
user nginx-app;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent"';
    
    access_log /var/log/nginx/access.log main;
    sendfile on;
    keepalive_timeout 65;
    gzip on;
    
    include /etc/nginx/conf.d/*.conf;
}
```

---

## MySQL Database

```dockerfile
# ============================================
# MySQL - Database Server
# ============================================
FROM mysql:8.0

LABEL maintainer="your-email@example.com"
LABEL description="MySQL database with custom configuration"

ENV MYSQL_ROOT_PASSWORD=changeme
ENV MYSQL_DATABASE=appdb
ENV MYSQL_USER=appuser
ENV MYSQL_PASSWORD=apppassword

# Custom MySQL configuration
COPY my.cnf /etc/mysql/conf.d/custom.cnf

# Initialization scripts (run on first startup)
COPY init-scripts/ /docker-entrypoint-initdb.d/

VOLUME /var/lib/mysql
EXPOSE 3306

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD mysqladmin ping -h localhost -u root -p${MYSQL_ROOT_PASSWORD} || exit 1
```

**Example my.cnf:**
```ini
[mysqld]
innodb_buffer_pool_size = 256M
innodb_log_file_size = 64M
max_connections = 200
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
slow_query_log = 1
long_query_time = 2
bind-address = 0.0.0.0
```

**Example init script (01-schema.sql):**
```sql
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## PostgreSQL Database

```dockerfile
# ============================================
# PostgreSQL - Database Server
# ============================================
FROM postgres:16-alpine

LABEL maintainer="your-email@example.com"

ENV POSTGRES_DB=appdb
ENV POSTGRES_USER=appuser
ENV POSTGRES_PASSWORD=apppassword

COPY postgresql.conf /etc/postgresql/
COPY init-scripts/ /docker-entrypoint-initdb.d/

VOLUME /var/lib/postgresql/data
EXPOSE 5432

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB} || exit 1
```

---

## Apache HTTP Server

```dockerfile
# ============================================
# Apache + PHP - Full Web Stack
# ============================================
FROM php:8.2-apache

LABEL maintainer="your-email@example.com"

# Install PHP extensions
RUN docker-php-ext-install pdo pdo_mysql mysqli opcache

# Install additional tools
RUN apt-get update && apt-get install -y \
    libzip-dev zip \
    && docker-php-ext-install zip \
    && rm -rf /var/lib/apt/lists/*

# Enable Apache modules
RUN a2enmod rewrite headers ssl

# Copy configuration
COPY apache-vhost.conf /etc/apache2/sites-available/000-default.conf
COPY php.ini /usr/local/etc/php/

# Copy application
COPY src/ /var/www/html/

# Set permissions
RUN chown -R www-data:www-data /var/www/html

EXPOSE 80 443

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD curl -f http://localhost/ || exit 1
```

---

## Grafana

```dockerfile
# ============================================
# Grafana - Monitoring Dashboard
# ============================================
FROM grafana/grafana:10.2.3

LABEL maintainer="your-email@example.com"

ENV GF_SECURITY_ADMIN_USER=admin
ENV GF_SECURITY_ADMIN_PASSWORD=admin
ENV GF_USERS_ALLOW_SIGN_UP=false
ENV GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-piechart-panel

COPY grafana.ini /etc/grafana/grafana.ini
COPY provisioning/datasources/ /etc/grafana/provisioning/datasources/
COPY provisioning/dashboards/ /etc/grafana/provisioning/dashboards/
COPY dashboards/ /var/lib/grafana/dashboards/

VOLUME /var/lib/grafana
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/api/health || exit 1
```

**Example datasource provisioning:**
```yaml
# provisioning/datasources/prometheus.yml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
```

---

## Kibana

```dockerfile
# ============================================
# Kibana - Elasticsearch Dashboard
# ============================================
FROM docker.elastic.co/kibana/kibana:8.11.3

LABEL maintainer="your-email@example.com"

ENV ELASTICSEARCH_HOSTS=http://elasticsearch:9200
ENV SERVER_NAME=kibana
ENV SERVER_HOST=0.0.0.0

COPY kibana.yml /usr/share/kibana/config/kibana.yml

EXPOSE 5601

HEALTHCHECK --interval=30s --timeout=10s --start-period=120s --retries=5 \
    CMD curl -f http://localhost:5601/api/status || exit 1
```

**Example kibana.yml:**
```yaml
server.name: kibana
server.host: "0.0.0.0"
server.port: 5601
elasticsearch.hosts: ["http://elasticsearch:9200"]
```

---

## Elasticsearch

```dockerfile
# ============================================
# Elasticsearch - Search Engine
# ============================================
FROM docker.elastic.co/elasticsearch/elasticsearch:8.11.3

ENV discovery.type=single-node
ENV ES_JAVA_OPTS="-Xms512m -Xmx512m"
ENV xpack.security.enabled=false

VOLUME /usr/share/elasticsearch/data
EXPOSE 9200 9300

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=5 \
    CMD curl -f http://localhost:9200/_cluster/health || exit 1
```

---

## Complete Stack (docker-compose.yml)

```yaml
version: '3.8'

services:
  nginx:
    build: ./nginx
    ports:
      - "80:80"
    depends_on:
      - app

  app:
    build: ./app
    environment:
      - DB_HOST=mysql
    depends_on:
      - mysql

  mysql:
    build: ./mysql
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: appdb
    volumes:
      - mysql_data:/var/lib/mysql

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.3
    environment:
      - discovery.type=single-node
    volumes:
      - es_data:/usr/share/elasticsearch/data

  kibana:
    build: ./kibana
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

  grafana:
    build: ./grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana

volumes:
  mysql_data:
  es_data:
  grafana_data:
```

---

*Part of the [Containers Documentation](README.md)*

