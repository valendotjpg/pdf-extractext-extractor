# Evidencia: corrida final

Dos corridas seguidas de k6 y Vegeta, con la versión final del servicio y la
configuración por defecto, en el mismo equipo que el [informe](../informe-carga.md#2-entorno-de-medición).
Cada carpeta tiene los reportes tal como los generan los scripts.

| | |
|---|---|
| Fecha | 07/10/2026, 23:30 a 23:41 (UTC−3) |
| Commit | Corrida 1: `8e499a1`. Corrida 2: `c50822c` (sólo mueve las pruebas a `tests/stress/`; el servicio es el mismo) |
| Comandos | `docker compose up --build -d`, luego `docker compose run --rm --service-ports k6` y `docker compose run --rm vegeta` |
| Configuración | `EXTRACTOR_REPLICAS=5`, `QUEUE_TIMEOUT_S=20` (valores por defecto, sin `.env`) |
| PDFs | Set oficial, verificado por SHA-256 ([lista y hashes](../../tests/stress/README.md#pdfs-de-prueba)) |
| Entorno | Intel Core i5-8365U (4 núcleos / 8 hilos), Docker Desktop 29.8.2, Compose 5.5.1 |

## k6, spike

| Corrida | Peticiones | req/s | Error | p50 | p90 | p95 | Máx | Reportes |
|---|---|---|---|---|---|---|---|---|
| Profesor | 1.037 | 25,35 | 0,00 % | 1,88 s | 7,83 s | 8,80 s | 13,94 s | — |
| 1 | 652 | 15,69 | 0,00 % | 5,38 s | 7,18 s | 7,41 s | 8,31 s | [json](corrida-1/k6-spike.json), [html](corrida-1/k6-spike.html) |
| 2 | 479 | 11,27 | 0,00 % | 7,99 s | 9,52 s | 9,73 s | 10,79 s | [json](corrida-2/k6-spike.json), [html](corrida-2/k6-spike.html) |

## Vegeta, 50 req/s durante 30 s

| Corrida | req/s efectivas | Éxito | Timeouts (código 0) | 503 | p50 | Reportes |
|---|---|---|---|---|---|---|
| Profesor | 16,65 | 66,53 % | 501 (33,40 %) | — | 14,89 s | — |
| 1 | 4,05 | 14,33 % (215) | 16 (1,07 %) | 1.269 | 20,01 s | [json](corrida-1/vegeta-report.json), [gráfico](corrida-1/vegeta-plot.html) |
| 2 | 3,98 | 13,93 % (209) | 11 (0,73 %) | 1.280 | 20,01 s | [json](corrida-2/vegeta-report.json), [gráfico](corrida-2/vegeta-plot.html) |

Los resultados coinciden con los del informe: en k6 ninguna petición falla, aunque el
throughput varía entre corridas, como ya se había visto en el Experimento 4. En Vegeta, casi
no hay timeouts: lo que no se puede atender recibe 503 en vez de colgarse hasta los 30 s.
Por qué no se alcanza al profesor en este hardware está en la
[sección 5 del informe](../informe-carga.md#5-cuello-de-botella-y-conclusiones).

Para reproducir: copiar los PDFs en `tests/stress/pdfs/`, correr los comandos de arriba y
comparar con `tests/stress/results/`.
