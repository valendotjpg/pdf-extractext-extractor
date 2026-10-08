# Pruebas de carga y estrés

Scripts para medir el extractor-service con los perfiles de la cátedra.
Todo corre en Docker: no hace falta instalar k6 ni Vegeta.

```
tests/stress/
├── pdfs/      los 4 PDFs oficiales de la cátedra (no se versionan)
├── k6/        prueba spike, modelo cerrado
├── vegeta/    prueba de carga fija, modelo abierto
└── results/   salida de cada corrida (ignorada por git)
```

## PDFs de prueba

Son material de terceros, así que no están en el repo. Antes de correr las pruebas hay
que copiar en `tests/stress/pdfs/` los 4 PDFs oficiales que provee la cátedra, con estos
nombres exactos (los usan los scripts):

| Archivo | Tamaño | SHA-256 |
|---|---|---|
| `2020-Scrum-Guide-Spanish-Latin-South-American.pdf` | 319.343 B | `6bbff3f361e52016cbb75a72671f1b31b5e043fdc1cc3f022f1b15a32795a256` |
| `Essential-Kanban-Condensed-Spanish.pdf` | 8.901.466 B | `257f73b908420ac5c78852fb8673beb683a8ca0c05d2e6a82b5e9629f6ad7be8` |
| `Filosofia Lean.pdf` | 668.142 B | `c77b55bd68477ecc0d96074ea4606bffe5802a1c8a241a725f6eb358be17ce40` |
| `scrum_manager_historias_usuario.pdf` | 3.830.899 B | `30efd6fcd0c022665676edcf164feb28e6b43dcce53ab9738550b99d70f5fd84` |

Son los PDFs con los que se midieron los resultados del informe (desde el
Experimento 3) y la [evidencia final](../../docs/evidencia/README.md). Para confirmar
que se usa el mismo set:

```bash
sha256sum tests/stress/pdfs/*.pdf
```

En PowerShell: `Get-FileHash tests/stress/pdfs/*.pdf`.

Igual que los scripts del profesor, el PDF va crudo en el body con
`Content-Type: application/pdf`. Desde el host, el extractor responde en
`http://localhost:8080/extract` (el puerto se cambia con `EXTRACTOR_PORT`).

## Cómo correrlas

Desde la raíz del repo, con el sistema levantado (`docker compose up --build -d`):

```bash
docker compose run --rm --service-ports k6
docker compose run --rm vegeta
```

| Prueba | Perfil | Resultados en `results/` |
|---|---|---|
| k6 (spike) | 0 → 100 VUs en 10 s, 20 s sostenidos, bajada a 0 en 10 s | `k6-spike.json`, `k6-spike.html` |
| Vegeta (carga fija) | 50 req/s durante 30 s, timeout 30 s | `vegeta-report.json`, `vegeta-plot.html` |

Mientras corre k6, el dashboard en vivo está en http://localhost:5665.

Vegeta acepta `RATE`, `DURATION` y `TIMEOUT` como variables de entorno,
por ejemplo `docker compose run --rm -e RATE=25 vegeta`. Del lado del servicio se
pueden variar la cantidad de réplicas (`EXTRACTOR_REPLICAS`) y la espera máxima antes
del 503 (`QUEUE_TIMEOUT_S`), por ejemplo `EXTRACTOR_REPLICAS=1 docker compose up -d`.
