---
title: "Introducción"
---

Bienvenido a la documentación del proyecto **TicTacToe API**, un sistema que combina:

- **Flask + Waitress** como servidor backend.
- **Docker + Docker Compose** para encapsular la aplicación.
- **Nginx** como reverse proxy y balanceador de carga.
- **Hugo** para documentar todo el proceso.

El objetivo de este proyecto es ejecutar una API de forma escalable desde múltiples dispositivos, aprovechando la potencia de Docker y la estabilidad de Waitress como servidor WSGI.

Aquí aprenderás:

1. Cómo instalar Waitress en un entorno Python.
2. Cómo configurarlo para servir tu API Flask.
3. Cómo crear una infraestructura con varios contenedores corriendo instancias idénticas.
4. Cómo exponer todo esto a internet a través de Nginx con balanceo de carga.

Al final tendrás una arquitectura estable, escalable y fácil de replicar.