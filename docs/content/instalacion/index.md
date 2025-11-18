---
title: "Instalación Básica de Waitress"
---

En esta sección veremos cómo se instala **Waitress** dentro del contenedor Docker que ejecuta la API.  
Waitress es el servidor WSGI que utilizamos para servir la aplicación Flask en producción.

---

## 1. Instalación de Waitress mediante Docker

En este proyecto, Waitress **no se instala manualmente en el sistema**, sino que se incluye directamente dentro del **Dockerfile** del servicio API.

Esto garantiza:

- Un entorno limpio y reproducible  
- Independencia del sistema operativo  
- Misma configuración en todas las instancias (tictactoe1, tictactoe2, etc.)  

---

## 2. Dockerfile con instalación automática de dependencias

El paquete Waitress se instala dentro del contenedor utilizando `pip install`.  
Este es el bloque relevante del Dockerfile:

```dockerfile
RUN pip install --upgrade pip setuptools wheel && \
    pip install waitress && \
    pip install -e .
```

Aquí ocurre lo siguiente:

- Se actualiza pip y herramientas base  
- Se instala **Waitress**  
- Se instalan las dependencias del proyecto (definidas en `pyproject.toml`)  

---

## 3. Ejecución de la API con Waitress dentro del contenedor

Waitress se utiliza como **entrypoint** del contenedor:

```dockerfile
CMD ["waitress-serve", "--listen=0.0.0.0:8000", "main:app"]
```

Esto significa:

- La API escucha en el puerto `8000` dentro del contenedor  
- Waitress sirve la aplicación Flask disponible en `main.py` bajo el nombre `app`  
- Todo está totalmente automatizado con Docker  

---

## 4. Levantar la aplicación ya configurada con Waitress

Para construir e iniciar los contenedores:

```bash
docker-compose build
docker-compose up -d
```

Esto lanza:

- Dos instancias de la API (`tictactoe1` y `tictactoe2`)
- Cada una ejecutada por Waitress
- Nginx como reverse proxy sobre ellas

---

## Resultado final

A diferencia de una instalación manual en el sistema, aquí:

- No es necesario configurar entornos virtuales  
- No debes instalar Waitress en tu máquina  
- No debes ejecutar `waitress-serve` tú mismo  

**Todo ocurre automáticamente dentro de los contenedores**, garantizando un entorno limpio, escalable y replicable.