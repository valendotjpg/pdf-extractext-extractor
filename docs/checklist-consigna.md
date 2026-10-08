# Checklist contra la consigna del TP

Cada punto del TP "Test de Carga, Estrés y Optimización de Microservicio", con dónde se
cumple en este repositorio. ✅ cumplido · ⚠️ cumplido con una salvedad.

## Requerimientos del microservicio

| Punto | Estado | Evidencia |
|---|---|---|
| Endpoint `POST /extract` | ✅ | [`extractor_service/main.py`](../extractor_service/main.py), [contrato](extractor_contract.md) |
| Entrada: PDF binario en el body | ✅ | PDF crudo con `Content-Type: application/pdf`, como los scripts del profesor ([contrato](extractor_contract.md#request)) |
| Salida: 200 JSON con `content` y `page_count` | ✅ | [contrato](extractor_contract.md#respuesta-exitosa-200), [`tests/test_api.py`](../tests/test_api.py) |
| `content` en Markdown | ✅ | Títulos, subtítulos, negritas y párrafos ([`pdf_extractor.py`](../extractor_service/pdf_extractor.py), [`tests/test_pdf_extractor.py`](../tests/test_pdf_extractor.py)) |

## Condiciones del entorno

| Punto | Estado | Evidencia |
|---|---|---|
| 1. Imagen Docker y despliegue con docker compose | ✅ | [`Dockerfile`](../Dockerfile), [`docker-compose.yml`](../docker-compose.yml) |
| 2. Hasta 5 réplicas | ✅ | `deploy.replicas: ${EXTRACTOR_REPLICAS:-5}` detrás de nginx |
| 3. Límite explícito de CPU y RAM en cada contenedor | ✅ | Réplicas 1.0 CPU / 1 GB, nginx 1.0 CPU / 512 MB. k6 y Vegeta no tienen límites porque son el generador de carga, no el sistema medido (comentado en el compose) |
| 4. Set oficial en `tests/stress/pdfs` | ✅ | No se versiona por ser material de terceros; nombres y SHA-256 en [`tests/stress/README.md`](../tests/stress/README.md#pdfs-de-prueba) |

## Pruebas y benchmark

| Punto | Estado | Evidencia |
|---|---|---|
| k6 spike: 0 → 100 VUs en 10 s, 20 s sostenidos, bajada en 10 s | ✅ | [`tests/stress/k6/spike.js`](../tests/stress/k6/spike.js) |
| Vegeta: 50 req/s durante 30 s, timeout 30 s, rotando los 4 PDFs | ✅ | [`tests/stress/vegeta/`](../tests/stress/vegeta/) |
| Superar al profesor | ⚠️ | No en este hardware: k6 11 a 16 req/s contra 25,35, con 0 % de errores. En Vegeta, ~1 % de timeouts contra 33,4 %, pero 14 % de éxito contra 66,5 %. El análisis está en el [informe, sección 5](informe-carga.md#5-cuello-de-botella-y-conclusiones) y los reportes en [evidencia](evidencia/README.md) |

## Pautas de optimización

| Pauta | Estado | Evidencia |
|---|---|---|
| Extracción eficiente | ✅ | PyMuPDF + Markdown propio: 177 ms por PDF contra 3,6 s de pypdf ([Experimento 3](informe-carga.md#experimento-3-medición-con-los-pdfs-oficiales)). El body se lee en memoria con tope de tamaño, y nginx no lo escribe a disco |
| Backpressure: 503 controlado | ✅ | [`admission.py`](../extractor_service/admission.py); 503 tras `QUEUE_TIMEOUT_S` ([Experimento 1](informe-carga.md#experimento-1-backpressure-pool-de-procesos--503-por-saturación)) |
| 5 réplicas detrás de un reverse proxy | ✅ | nginx con `least_conn` ([`nginx/nginx.conf`](../nginx/nginx.conf), [Experimento 0](informe-carga.md#experimento-0-techo-de-la-infraestructura-stub-sin-procesamiento)) |
| Pool de workers separado del runtime HTTP | ✅ | `ProcessPoolExecutor` en [`main.py`](../extractor_service/main.py) |

## Evaluación cualitativa

| Punto | Estado | Evidencia |
|---|---|---|
| Twelve-Factor: config por entorno, port binding, sin estado, logs a stdout | ✅ | [`config.py`](../extractor_service/config.py), [`.env.example`](../.env.example), puerto 8000, sin estado entre peticiones, logs de uvicorn a stdout |
| Código modular y legible | ✅ | Responsabilidades separadas: HTTP (`main.py`), admisión (`admission.py`), extracción (`pdf_extractor.py`), configuración (`config.py`); 21 tests |
| Informe: arquitectura y decisiones | ✅ | [Informe, sección 1](informe-carga.md#1-arquitectura) |
| Informe: antes vs. después | ✅ | [Informe, sección 3](informe-carga.md#3-resultados) |
| Informe: proceso de investigación | ✅ | [Informe, sección 4](informe-carga.md#4-proceso-de-investigación), un experimento por cambio |

## Entregables

| Entregable | Estado | Evidencia |
|---|---|---|
| Repositorio con Dockerfile y compose reproducible con `docker compose up --build` | ✅ | Repositorio propio del microservicio; ver [README](../README.md#ejecución) |
| Script de k6 (spike) | ✅ | [`tests/stress/k6/spike.js`](../tests/stress/k6/spike.js) |
| Script de Vegeta (carga fija) | ✅ | [`tests/stress/vegeta/run.sh`](../tests/stress/vegeta/run.sh), [`targets.txt`](../tests/stress/vegeta/targets.txt) |
| Informe en Markdown con arquitectura y cuello de botella | ✅ | [`docs/informe-carga.md`](informe-carga.md) |
