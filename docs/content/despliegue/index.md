---
title: "Despliegue Final con Nginx y Waitress (Múltiples Contenedores)"
---

En esta sección verás el **despliegue completo en producción** utilizando:

- **Flask** + **Waitress** (backend WSGI)
- **Dockerfile** (para build de la API)
- **Docker Compose** (para levantar varias instancias)
- **Nginx** como reverse proxy y balanceador de carga
- **Múltiples contenedores** ejecutando la misma API
- **Hugo** para documentar el proyecto

Esta arquitectura permite que la API sea accedida desde múltiples dispositivos de forma estable y escalable.

---

# 1. Estructura del proyecto

Tu proyecto está organizado así:

```
SERVIDOR-WAITRESS/
 ├─ api/
 │   ├─ Dockerfile
 │   ├─ main.py
 │   ├─ pyproject.toml
 │   └─ uv.lock
 ├─ docs/   ← sitio Hugo para documentación
 ├─ nginx/
 │   └─ nginx.conf
 └─ docker-compose.yml
```

---

# 2. Dockerfile de la API

El Dockerfile se encarga de:

- Instalar dependencias
- Copiar el código
- Lanzar **Waitress** en el puerto `8000`

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY pyproject.toml ./

RUN pip install --upgrade pip setuptools wheel &&     pip install waitress &&     pip install -e .

COPY . .

EXPOSE 8000

CMD ["waitress-serve", "--listen=0.0.0.0:8000", "main:app"]
```

---

# 3. docker-compose.yml con múltiples instancias

Aquí levantas **dos contenedores idénticos** `tictactoe1` y `tictactoe2`, ambos corriendo la API por Waitress.

Luego Nginx se encarga del balanceo.

```yaml
version: "3.8"

services:
  tictactoe1:
    build: ./api
    container_name: tictactoe1
    restart: always
    networks:
      - tictactoe-net
    expose:
      - "8000"

  tictactoe2:
    build: ./api
    container_name: tictactoe2
    restart: always
    networks:
      - tictactoe-net
    expose:
      - "8000"

  nginx:
    image: nginx:latest
    container_name: nginx_tictactoe
    restart: always
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "9000:80"
    depends_on:
      - tictactoe1
      - tictactoe2
    networks:
      - tictactoe-net

networks:
  tictactoe-net:
    driver: bridge
```

---

# 4. Configuración de Nginx (nginx.conf)

Nginx actúa como:

- Reverse Proxy  
- Balanceador de carga  
- Punto de entrada externo (puerto 9000)  

```nginx
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    upstream tictactoe_upstream {
        ip_hash;
        server tictactoe1:8000;
        server tictactoe2:8000;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://tictactoe_upstream;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

`ip_hash` asegura que cada cliente siempre vaya al mismo backend (ideal para sesiones persistentes).

---

# 5. Levantar todo el sistema

Desde la carpeta raíz del proyecto:

```bash
docker-compose build
docker-compose up -d
```

Comprobar contenedores:

```bash
docker ps
```

Todos deben aparecer como **Up**.

---

# 6. Probar la API desde el navegador o curl

Registrar un device:

```bash
curl -X POST http://localhost:9000/devices
```

Ver dispositivos:

```bash
curl http://localhost:9000/devices
```

Nginx distribuirá las peticiones entre `tictactoe1` y `tictactoe2`.

---

# 7. Confirmar balanceo de carga

Ver logs:

```bash
docker logs tictactoe1
docker logs tictactoe2
```

Ambos deben recibir tráfico.

---