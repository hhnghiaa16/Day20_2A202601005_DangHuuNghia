# Reflection — Lab 20 (Personal Report)

**Họ Tên:** Đặng Hữu Nghĩa - 2A202601005

**Cohort:** _A20_K2

**Ngày submit:** _2026-06-24_

---

## 1. Hardware spec (từ `00-setup/detect-hardware.py`)

> Paste output của `python 00-setup/detect-hardware.py` vào đây, hoặc điền thủ công:

- **OS:** _Windows_
- **CPU:** _11th Gen Intel(R) Core(TM) i5-11400H @ 2.70GHz_
- **Cores:** _6 physical / 12 logical_
- **CPU extensions:** _AMD64_
- **RAM:** _7.7 GB_
- **Accelerator:** _NVIDIA GeForce GTX 1650 with Max-Q Design, 4096 MiB_
- **llama.cpp backend đã chọn:** _CUDA_
- **Recommended model tier:** _TinyLlama-1.1B (Q4_K_M)_

**Setup story** (≤ 80 chữ): những gì cần thay đổi để lab chạy được trên máy bạn (vd: dùng WSL2, install CUDA Toolkit, fall back sang Vulkan vì ROCm phiên bản kén, tắt antivirus để pip install nhanh hơn, v.v.):

>_Máy Windows gặp lỗi khi cài `llama-cpp-python`, chủ yếu do đường dẫn dài và build tool. Sau khi sửa cách cài package, đã tạo được `hardware.json`, tải TinyLlama và chạy benchmark/server thành công._

---

## 2. Track 01 — Quickstart numbers (từ `benchmarks/01-quickstart-results.md`)

> Paste bảng từ `benchmarks/01-quickstart-results.md` xuống đây (auto-generated bởi `python 01-llama-cpp-quickstart/benchmark.py`).

| Model | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode rate (tok/s) |
|---|--:|--:|--:|--:|--:|
| tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf | 1621 | 85 / 122 | 34.0 / 39.0 | 2145 / 2544 / 2612 | 29.4 |
| tinyllama-1.1b-chat-v1.0.Q2_K.gguf | 489 | 129 / 156 | 25.2 / 27.9 | 1246 / 1778 / 1812 | 39.6 |

**Một quan sát** (≤ 50 chữ): Q4_K_M vs Q2_K trên máy bạn — số liệu nói gì? Quality đáng đánh đổi không?

>_Q2_K load nhanh hơn và decode nhanh hơn Q4_K_M, nhưng Q4_K_M hợp lý hơn nếu cần chất lượng câu trả lời ổn định._

---

## 3. Track 02 — llama-server load test

> Chạy 2 lần locust ở concurrency 10 và 50, paste tóm tắt bên dưới.

| Concurrency | Total RPS | TTFB P50 (ms) | E2E P95 (ms) | E2E P99 (ms) | Failures |
|--:|--:|--:|--:|--:|--:|
| 10 | 0.27 | 26000 | 39000 | 39000 | 0 |
| 50 | 0.35 | 21000 | 41000 | 41000 | 0 |

>_Đã dùng kết quả locust làm quan sát chính. Khi tăng concurrency từ 10 lên 50, P95 tăng từ 39s lên 41s, nghĩa là server bắt đầu nghẽn ở bước sinh token._

---

## 4. Track 03 — Milestone integration

- **N16 (Cloud/IaC):** _stub: chạy local llama-server trên `localhost:8080`_
- **N17 (Data pipeline):** _stub: dữ liệu mẫu nằm trong `TOY_DOCS`_
- **N18 (Lakehouse):** _stub: chưa dùng Delta/Iceberg, dùng in-memory data_
- **N19 (Vector + Feature Store):** _stub: keyword retrieval từ `TOY_DOCS`_

**Nơi tốn nhiều ms nhất** trong pipeline (đo bằng `time.perf_counter` trong `pipeline.py`):

- embed: _0 ms (stub, chưa dùng embedder riêng)_
- retrieve: _0.0–0.3 ms_
- llama-server: _5455.3–8216.4 ms_

**Reflection** (≤ 60 chữ): bottleneck nằm ở đâu? Có khớp với kỳ vọng không?

_Bottleneck nằm gần như hoàn toàn ở llama-server. Retrieve đang là stub rất nhẹ, còn thời gian chính là prefill/decode khi gọi `/v1/chat/completions`._

---

## 5. Bonus — The single change that mattered most

> **Most important section.** Pick **một** thay đổi từ bonus track (build flag, thread sweep, quant pick, GPU offload, KV-cache quantization, speculative decoding, bất cứ challenge nào trong `BONUS-llama-cpp-optimization/CHALLENGES.md`) đã tạo ra speedup lớn nhất trên máy bạn.

**Change:** _Single change từ Track 01: đổi quantization từ Q4_K_M sang Q2_K để giảm kích thước model và tăng tốc decode. Đây không phải bonus sweep riêng._

**Before vs after** (paste 2-3 dòng từ sweep output):

```
before: Q4_K_M decode rate = 29.4 tok/s, E2E P50 = 2145 ms
after:  Q2_K decode rate = 39.6 tok/s, E2E P50 = 1246 ms
speedup: ~1.35× decode rate, ~1.72× E2E P50
```

**Tại sao nó work** (1–2 đoạn ngắn — đây là phần grader đọc kỹ nhất):

_Q2_K nén model nhỏ hơn nên load nhanh hơn và giảm áp lực memory bandwidth khi decode. Trên máy RAM 7.7 GB, điều này giúp latency E2E giảm rõ. Đổi lại, Q2_K có thể làm chất lượng câu trả lời kém ổn định hơn Q4_K_M, nên nếu ưu tiên chất lượng thì Q4_K_M vẫn là lựa chọn an toàn hơn._

---

## 6. (Optional) Điều ngạc nhiên nhất

_(1–2 câu — không bắt buộc, nhưng người grader đọc tất cả)_

_Concurrency cao hơn không làm latency tốt hơn. Khi nhiều request cùng lúc, server vẫn bị giới hạn bởi tốc độ sinh token của model._

---

## 7. Self-graded checklist

- [x] `hardware.json` đã commit
- [x] `models/active.json` đã commit (hoặc paste path snapshot vào section 1)
- [x] `benchmarks/01-quickstart-results.md` đã commit
- [x] `benchmarks/02-server-results.md` (hoặc CSV từ `record-metrics.py`) đã commit
- [ ] `benchmarks/bonus-*.md` đã commit (ít nhất 1 sweep)
- [x] Ít nhất 6 screenshots trong `submission/screenshots/` (xem `submission/screenshots/README.md`)
- [x] `make verify` exit 0 (chạy ngay trước khi push)
- [ ] Repo trên GitHub ở chế độ **public**
- [ ] Đã paste public repo URL vào VinUni LMS

---

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Nếu private, grader không xem được → 0 điểm.
