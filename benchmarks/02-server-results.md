# 02 - llama-server load test results

Server: `llama-cpp-python` OpenAI-compatible server on `http://localhost:8080`.

| Concurrency | Total RPS | P50 (ms) | P95 (ms) | P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 0.27 | 26000 | 39000 | 39000 | 0 |
| 50 | 0.35 | 21000 | 41000 | 41000 | 0 |

Notes:

- Evidence screenshots are in `submission/screenshots/04-locust-10.png` and `submission/screenshots/05-locust-50.png`.
- Native `/metrics` was not captured in this run, so batching observation is based on locust latency under load.
