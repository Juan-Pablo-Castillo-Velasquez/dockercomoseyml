```yaml
version: '3.9'

services:
  db:
    image: mysql:8.0
    container_name: mysql_db
    environment:
      MYSQL_ROOT_PASSWORD: juan123
      MYSQL_DATABASE: test_db
      MYSQL_USER: juanpablo
      MYSQL_PASSWORD: juan123
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - web_network

  web:
    image: php:8-apache
    container_name: php_web
    depends_on:
      - db
    volumes:
      - ./:/var/www/html/    # monta tu carpeta view/
    ports:
      - "8100:80"
    networks:
      - web_network
    stdin_open: true
    tty: true
    command: bash -c "apache2-foreground"  # inicia Apache

  node:
    image: node:20-alpine
    container_name: node_app
    working_dir: /app
    volumes:
      - ./node/:/app
    ports:
      - "3000:3000"
    command: sh -c "npm install && npm start"
    depends_on:
      - db
    networks:
      - web_network

volumes:
  db_data:

networks:
  web_network:
    driver: bridge
