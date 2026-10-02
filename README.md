# HDT 4: suites de Bruno contra un API

Asigna: jueves 24 de septiembre.
Entrega: jueves 1 de octubre, 23:59.
## El API

```bash
uv run api.py                      # http://127.0.0.1:8000
DEMO_API_KEY=s3cret uv run api.py  # las escrituras piden X-API-Key
```

El estado vive en memoria. `POST /reset` lo devuelve a los tres libros iniciales. Cada vez que apagas el API se resetea. 

| Método | Ruta | Éxito | Error |
|---|---|---|---|
| GET | `/health` | 200 | |
| GET | `/books?author=` | 200 | |
| POST | `/books` | 201 + `Location` | 422 |
| GET | `/books/{id}` | 200 | 404 |
| PUT | `/books/{id}` | 200 | 404, 422 |
| PATCH | `/books/{id}` | 200 | 404, 422 |
| DELETE | `/books/{id}` | 204 | 404 |
| HEAD | `/books/{id}` | 200 | 404 |
| OPTIONS | `/books` | 204 + `Allow` | |
| POST | `/reset` | 200 | |

## Especificación

| Campo o regla | Valor |
|---|---|
| `title`, `author` | obligatorios, de 1 a 120 caracteres |
| `year` | obligatorio, de 1450 a 2100 |
| `copies` | de 0 a 999, por defecto 1 |
| `?author=` | subcadena, sin distinguir mayúsculas |
| `PUT` | reemplaza el libro completo |
| `PATCH` | cambia solo los campos enviados |
| Con `DEMO_API_KEY` definida | POST, PUT, PATCH y DELETE sin la llave correcta dan 401 |
| Sin `DEMO_API_KEY` | no hay autenticación |

## Qué se entrega

Una carpeta `bruno/` con las 2 colecciones, cada una con su `bruno.json` y un environment `local`:

| Colección | Contenido |
|---|---|
| `smoke/` | 3 a 5 requests. El servicio responde y los caminos principales viven. |
| `regresion/` | Entre 10 y 15 requests diseñados con técnicas de ISTQB. |

Reglas para las tres:

- Cada colección abre con `POST /reset` y pasa corrida dos veces seguidas.
- Cada request lleva al menos una aserción, en `assert` o en `tests`.
- En la regresion poner con un comment la técnica de testing de ISTQB utilizada. 

