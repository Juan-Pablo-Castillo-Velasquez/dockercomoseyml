# docker-compose.yml para producción básica
version: '3.9'

services:

  # Base de datos MySQL
  db:
    image: mysql:8.0             # Imagen oficial de MySQL
    container_name: mysql_db
    environment:
      # Variables de entorno desde archivo .env para seguridad
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    # No exponemos puerto en producción, solo red interna
    # ports:
    #   - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql   # Volumen persistente
    networks:
      - web_network
    healthcheck:                 # Verifica que MySQL esté listo
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Servicio web PHP con Apache
  web:
    build: ./web                 # Construye imagen desde Dockerfile
    container_name: php_web
    depends_on:
      - db
    networks:
      - web_network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 30s
      timeout: 10s
      retries: 3
    # En producción no montamos volumen de código local
    # volumes:
    #   - ./:/var/www/html
    # ports serán manejados por un reverse proxy
    # ports:
    #   - "8100:80"

  # Servicio Node.js
  node:
    build: ./node               # Construye imagen desde Dockerfile
    container_name: node_app
    working_dir: /app
    depends_on:
      - db
    networks:
      - web_network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/"]
      interval: 30s
      timeout: 10s
      retries: 3
    # Volumen para desarrollo:
    # volumes:
    #   - ./node/:/app
    # Ports se manejan por reverse proxy
    # ports:
    #   - "3000:3000"

  # Reverse proxy Nginx (opcional, recomendado para producción)
  proxy:
    image: nginx:alpine
    container_name: nginx_proxy
    depends_on:
      - web
      - node
    networks:
      - web_network
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro   # SSL si se usa
    ports:
      - "80:80"
      - "443:443"
    restart: always

# Volúmenes persistentes
volumes:
  db_data:

# Redes internas
networks:
  web_network:
    driver: bridge
