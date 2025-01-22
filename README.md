# app-docker-nginx
This repository builds an app-react with api-node using Docker and NGINX.

## PASO 1: Añadir NGINX AL DOCKER-COMPOSE


services:
  react-app:
    build:
      context: ./react/next-productivity-app
        # Name of the dockerfile
      dockerfile: Dockerfile
    container_name: react-app
    environment:
      - REACT_APP_BASE_URL=https://172.50.40.218/tasks
    ports:
     # Host port:Container port
      - '3000:3000'
    stdin_open: true
  # Add the node-js service
  api-node:
  # Location to the node.js dockerfile
    build:
      context: ./node/next-productivity-API
      # Name of the dockerfile
      dockerfile: Dockerfile
    container_name: api-node
    ports:
      - '5000:5000'
    volumes:
      - ${PWD}/next-productivity-API/tasks.json:/data/tasks.j
  nginx:
    image: nginx:alpine
    container_name: nginx_proxy
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/certs:/etc/nginx/certs
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - react-app
      - api-node


## Paso 2: Crear una carpeta `nginx`

Crea una carpeta llamada `nginx` en tu proyecto y dentro de ella, un archivo **importantísimo** llamado `nginx.conf` con el siguiente contenido:

```nginx
events {}
http {
    # Configura el servidor que escucha en HTTP (puerto 80)
    server {
        listen 80;
        server_name _;

        # Redirige todas las solicitudes HTTP a HTTPS
        return 301 https://$host$request_uri;
    }

    # Configura el servidor HTTPS (puerto 443)
    server {
        listen 443 ssl;
        server_name _;

        # Configuración de SSL
        ssl_certificate /etc/nginx/certs/selfsigned.crt;
        ssl_certificate_key /etc/nginx/certs/selfsigned.key;

        # Ajustes de SSL recomendados
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        # ssl_prefer_server_ciphers on;

        location / {
            proxy_pass http://react-app:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location /tasks/ {
            proxy_pass http://api-node:5000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            # Configura CORS aquí
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
            add_header Access-Control-Allow-Headers "Origin, Authorization, Content-Type, Accept";
        }
    }
}



  

