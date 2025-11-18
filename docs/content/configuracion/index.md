---
title: "Configuración de Waitress"
---

En esta sección veremos cómo ajustar Waitress para mejorar rendimiento y estabilidad en producción.

## 1. Ejecutar la aplicación con parámetros

```bash
waitress-serve \
  --listen=0.0.0.0:8000 \
  --threads=8 \
  --asyncore-use-poll \
  main:app
```

## 2. Parámetros recomendados

| Parámetro | Descripción |
|----------|-------------|
| `--threads` | Número de threads por worker. 4–8 suele ser suficiente. |
| `--listen` | Dirección IP y puerto donde escuchar. |
| `--asyncore-use-poll` | Mayor compatibilidad en contenedores. |

## 3. Uso desde Docker

```dockerfile
CMD ["waitress-serve", "--listen=0.0.0.0:8000", "main:app"]
```

Waitress funciona perfectamente dentro de contenedores debido a su simplicidad y estabilidad.

## 4. Logs

Para ver logs en un contenedor:

```bash
docker logs api1
```

Esto te permite verificar la actividad y detectar errores.