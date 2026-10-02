# Técnicas aplicadas en `regresion/`

Todas las filas fueron corridas contra el API real (`api.py`) con `DEMO_API_KEY=s3cret`.
Los resultados obtenidos coinciden con lo esperado según la especificación — ningún hallazgo
fuera de la tabla del enunciado.

| # | Request | Técnica | Valor probado | Resultado esperado | Resultado obtenido |
|---|---|---|---|---|---|
| 01 | POST valido | Partición de equivalencia (clase válida) | title/author/year/copies todos dentro de rango | 201 | 201 ✓ |
| 02 | POST title vacío | Partición de equivalencia (clase inválida) | `title=""` (bajo el mínimo de 1 carácter) | 422 | 422 ✓ |
| 03 | year=1450 | Valores frontera | year en el mínimo permitido | 201 | 201 ✓ |
| 04 | year=1449 | Valores frontera | year un valor bajo el mínimo | 422 | 422 ✓ |
| 05 | year=2100 | Valores frontera | year en el máximo permitido | 201 | 201 ✓ |
| 06 | year=2101 | Valores frontera | year un valor sobre el máximo | 422 | 422 ✓ |
| 07 | copies=1000 | Valores frontera | copies un valor sobre el máximo (999) | 422 | 422 ✓ |
| 08 | title de 121 caracteres | Valores frontera | title un carácter sobre el máximo (120) | 422 | 422 ✓ |
| 09 | POST sin X-API-Key | Tabla de decisión (DEMO_API_KEY activa × header ausente) | Escritura sin header, con auth activa | 401 | 401 ✓ |
| 10 | POST con llave incorrecta y body inválido a la vez | Tabla de decisión (llave incorrecta × body inválido) | X-API-Key incorrecta + title vacío | 401 | 401 ✓ |
| 11 | Crea el libro del flujo | Transición de estados (no existe → existe) | POST válido | 201 | 201 ✓ |
| 12 | PUT actualiza el libro | Transición de estados (existe → actualizado) | Reemplazo completo | 200 | 200 ✓ |
| 13 | DELETE el libro | Transición de estados (actualizado → borrado) | — | 204 | 204 ✓ |
| 14 | GET tras borrar | Transición de estados (borrado → no existe) | — | 404 | 404 ✓ |

## Hallazgo relevante que no estaba explícito en el enunciado

El request 10 confirma algo que la tabla de la especificación no aclara: cuando una escritura
falla **tanto** la autenticación como la validación del body al mismo tiempo (llave incorrecta +
`title` vacío), el API responde **401**, no 422. Esto indica que la dependencia `require_key` se
resuelve antes que la validación del modelo Pydantic del body. Se documenta acá porque es el tipo
de comportamiento que una tabla de decisión está pensada para sacar a la luz.

## Técnicas usadas

- Partición de equivalencia (requests 01–02)
- Valores frontera (requests 03–08)
- Tabla de decisión (requests 09–10)
- Transición de estados (requests 11–14)

Se cubrieron las cuatro técnicas mencionadas en el enunciado (se pedían al menos tres distintas).

## Nota sobre la colección `integracion/`

La tabla "Qué se entrega" del enunciado solo define `smoke/` y `regresion/`, pero la sección de
evidencia pide `junit-integracion.xml`. No se generó esa colección porque no hay una definición
de qué debe cubrir. Revisar con el profesor si falta una fila en la tabla original.

## Cómo se corrió

```bash
# Terminal 1
DEMO_API_KEY=s3cret uv run api.py

# Terminal 2, desde la raíz de cada colección
newman run smoke.postman_collection.json -e local.postman_environment.json \
  --reporters cli,junit --reporter-junit-export junit-smoke.xml

newman run regresion.postman_collection.json -e local.postman_environment.json \
  --reporters cli,junit --reporter-junit-export junit-regresion.xml
```

El reporter `junit` viene incluido en Newman (no existe un paquete `newman-reporter-junit`
separado en el registro de npm; instalar solo `npm install -g newman`).

La variable `apiKey` del entorno `local` debe setearse a `s3cret` (el mismo valor de
`DEMO_API_KEY`) antes de correr — no se dejó escrito en el `local.postman_environment.json`
del repo para no commitear la llave.