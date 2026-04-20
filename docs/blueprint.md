# Day 13 Observability Lab Report

> **Instruction**: Fill in all sections below. This report is designed to be parsed by an automated grading assistant. Ensure all tags (e.g., `[GROUP_NAME]`) are preserved.

## 1. Team Metadata
- [GROUP_NAME]: NHÓM 20.5
- [REPO_URL]: https://github.com/hanhieu/Day13-nhom20.5
- [MEMBERS]:
  - Member A: Phan Anh Khôi - 2A202600276 | Role: Logging & PII
  - Member B: Nguyễn Hữu Quang - 2A202600167 | Role: Tracing & Tags
  - Member C: Nguyễn Hữu Huy - 2A202600166 | Role: SLO & Alerts
  - Member D: Nguyễn Gia Bảo - 2A202600156 | Role: Load test + Incident injection
  - Member E: Hàn Quang Hiếu - 2A202600056 | Role: Dashboard + Evidence
  - Member F: Nguyễn Bình Thành - 2A202600138 | Role: Blueprint + Demo lead
---

## 2. Group Performance (Auto-Verified)
- [VALIDATE_LOGS_FINAL_SCORE]: 100/100
- [TOTAL_TRACES_COUNT]: 302
- [PII_LEAKS_FOUND]: 0

---

## 3. Technical Evidence (Group)

### 3.1 Logging & Tracing
- [EVIDENCE_CORRELATION_ID_SCREENSHOT]: screenshot/Dashboard langfuse.png; screenshot/Dashboard langfuse.png
- [EVIDENCE_PII_REDACTION_SCREENSHOT]: 
- [EVIDENCE_TRACE_WATERFALL_SCREENSHOT]: screenshot/Dashboard langfuse.png; screenshot/Dashboard langfuse.png
- [TRACE_WATERFALL_EXPLANATION]: The trace shows the complete request flow with OpenAI API integration, including RAG retrieval spans, LLM generation with real token usage, and proper correlation ID propagation. The waterfall demonstrates latency spikes during incident scenarios (rag_slow) where retrieval spans take 2.5+ seconds. Langfuse dashboard displays 302 traces with real OpenAI model costs and correlation IDs.

### 3.2 Dashboard & SLOs
- [DASHBOARD_6_PANELS_SCREENSHOT]: screenshot/Dashboard 6 panel.png
- [SLO_TABLES]:

**Baseline (Normal Operation):**

| SLI | Target | Window | Current Value | Status |
|---|---:|---|---:|:---:|
| Latency P95 | < 5000ms | 28d | 150ms | ✅ PASS |
| Error Rate | < 5.0% | 28d | 0% | ✅ PASS |
| Cost Budget | < $5.0/day | 1d | $0.30 | ✅ PASS |
| Quality Score | > 0.80 | 28d | 0.88 | ✅ PASS |

**Incident: rag_slow (Retrieval Latency Spike):**

| SLI | Target | Window | Current Value | Status |
|---|---:|---|---:|:---:|
| Latency P95 | < 5000ms | 28d | 12088ms | ❌ FAIL |
| Error Rate | < 5.0% | 28d | 0% | ✅ PASS |
| Cost Budget | < $5.0/day | 1d | $0.45 | ✅ PASS |
| Quality Score | > 0.80 | 28d | 0.85 | ✅ PASS |

**Incident: tool_fail (Vector Store Timeout):**

| SLI | Target | Window | Current Value | Status |
|---|---:|---|---:|:---:|
| Latency P95 | < 5000ms | 28d | 0ms | ✅ PASS |
| Error Rate | < 5.0% | 28d | 100% | ❌ FAIL |
| Cost Budget | < $5.0/day | 1d | $0.00 | ✅ PASS |
| Quality Score | > 0.80 | 28d | N/A | ⚠️ N/A |

**Incident: cost_spike (Token Usage Spike):**

| SLI | Target | Window | Current Value | Status |
|---|---:|---|---:|:---:|
| Latency P95 | < 5000ms | 28d | 150ms | ✅ PASS |
| Error Rate | < 5.0% | 28d | 0% | ✅ PASS |
| Cost Budget | < $5.0/day | 1d | $8.50 | ❌ FAIL |
| Quality Score | > 0.80 | 28d | 0.82 | ✅ PASS |

**Combined Incidents (rag_slow + cost_spike):**

| SLI | Target | Window | Current Value | Status |
|---|---:|---|---:|:---:|
| Latency P95 | < 5000ms | 28d | 10098ms | ❌ FAIL |
| Error Rate | < 5.0% | 28d | 0% | ✅ PASS |
| Cost Budget | < $5.0/day | 1d | $9.20 | ❌ FAIL |
| Quality Score | > 0.80 | 28d | 0.85 | ✅ PASS |

### 3.3 Alerts & Runbook
- [ALERT_RULES_SCREENSHOT]: 
- [SAMPLE_RUNBOOK_LINK]: docs/alerts.md#high-latency-p95

---

## 4. Incident Response (Group)
- [SCENARIO_NAME]: rag_slow
- [SYMPTOMS_OBSERVED]: Latency P95 increased from ~150ms to 8000ms+, response times exceeded 10+ seconds, dashboard showed red SLO indicators
- [ROOT_CAUSE_PROVED_BY]: Trace ID req-87600be9 shows 12088ms latency, Log line with correlation_id "req-3daee5a0" shows latency_ms: 8000
- [FIX_ACTION]: Disabled rag_slow incident via python scripts/inject_incident.py --scenario rag_slow --disable
- [PREVENTIVE_MEASURE]: Implement caching for RAG retrieval, add circuit breakers for slow external services, set up P95 latency alerts with 5000ms threshold

---

## 5. Individual Contributions & Evidence

### Phan Anh Khôi - 2A202600276 (Member A)
- [TASKS_COMPLETED]: Implemented Correlation IDs middleware, enriched structured logs with context, and built the PII scrubber for sensitive data directly inside `app/logging_config.py` and `app/pii.py`.
- [EVIDENCE_LINK]: (Provide Link to commit or PR for Logging)

### Nguyễn Hữu Quang - 2A202600167 (Member B)
- [TASKS_COMPLETED]: Instrumented Langfuse tracing using the `@observe` decorator, tracked pipeline inputs/outputs, and propagated tags and metadata across application spans.
- [EVIDENCE_LINK]: (Provide Link to commit or PR for Tracing)

### Nguyễn Hữu Huy - 2A202600166 (Member C)
- [TASKS_COMPLETED]: Configured SLO targets (`config/slo.yaml`) and Alerting rules (`config/alert_rules.yaml`). Drafted runbook documentation in `docs/alerts.md` for incident response.
- [EVIDENCE_LINK]: (Provide Link to commit or PR for SLO & Alerts)

### Nguyễn Gia Bảo - 2A202600156 (Member D)
- [TASKS_COMPLETED]:
  1. **Nâng cấp `scripts/load_test.py`**: Viết lại hoàn toàn load test script. Xây dựng `RequestResult` dataclass thu thập dữ liệu từng request (status, latency, tokens, cost, correlation_id) và `LoadTestReport` class tính toán thống kê tổng hợp (P50/P95/P99 latency, error rate, total cost, avg cost/req, token totals). Thêm các CLI arguments: `--rounds` (lặp queries N lần để tạo nhiều traces), `--delay` (khoảng cách giữa requests), `--concurrency` (chạy song song với ThreadPoolExecutor), `--base-url`. Sửa lỗi BASE_URL không đồng nhất giữa các scripts (8001→8000). Export `run_load_test()` dưới dạng hàm có return `LoadTestReport` để `incident_demo.py` tái sử dụng.
  2. **Nâng cấp `scripts/inject_incident.py`**: Thêm `--status` để hiển thị trạng thái ON/OFF của tất cả incidents, `--all` để bật/tắt cả 3 scenarios cùng lúc, `--base-url` và error handling (HTTPStatusError, ConnectError). Hiển thị kết quả xác nhận sau mỗi thao tác.
  3. **Tạo mới `scripts/incident_demo.py`**: Script tự động hóa toàn bộ quy trình Incident Response gồm 6 phases liên tục: (1) Baseline traffic → (2) Inject incident → (3) Traffic under incident → (4) Disable incident → (5) Recovery traffic → (6) In bảng so sánh Baseline vs Incident vs Recovery với delta %. Hỗ trợ `--scenario` (chạy 1 scenario), `--all` (chạy tuần tự cả 3: rag_slow, tool_fail, cost_spike). Tự động kiểm tra kết nối tới app trước khi bắt đầu, disable tất cả incidents sau khi hoàn tất. In Root Cause Hints giải thích cách xác định nguyên nhân cho từng scenario dựa trên Metrics → Traces → Logs.
  4. **Bổ sung `data/sample_queries.jsonl`**: Thêm 5 queries mới (u11–u15) tăng diversity (refund, latency percentile, monitoring pipeline, CCCD PII test, structured logging best practices). Tổng 15 queries/round đảm bảo vượt yêu cầu 10 traces.
  5. **Chạy full test cả 3 scenarios** (`rag_slow`, `tool_fail`, `cost_spike`) và thu thập dữ liệu before/after cho Section 4 — Incident Response.

  **Giải thích kỹ thuật — Cách tính Percentile (P95) trong `LoadTestReport`**:
  Thuật toán percentile hoạt động như sau: sắp xếp mảng latency tăng dần (sorted), tính index = `round((p/100) × N + 0.5) − 1`, clamp vào khoảng `[0, N−1]`. Ví dụ với N=15 values: P95 → index = round(0.95 × 15 + 0.5) − 1 = round(14.75) − 1 = 15 − 1 = 14 → lấy phần tử cuối cùng (giá trị lớn nhất). Với N=100 values: P95 → index 94 → phần tử thứ 95, nghĩa là 95% requests có latency thấp hơn hoặc bằng giá trị này. Đây là SLI chính được dùng trong SLO `Latency P95 < 3000ms`.

- [EVIDENCE_LINK]:
  - Commit chính: `054d673` — `feat: upgrade load test & incident injection scripts` (nhánh `roleD`)
  - PR #1 merged vào main: `6fc68a2` — `Merge pull request #1 from hanhieu/roleD`
  - Files đã thay đổi: `scripts/load_test.py`, `scripts/inject_incident.py`, `scripts/incident_demo.py` (new), `data/sample_queries.jsonl`

### Hàn Quang Hiếu - 2A202600056 (Member E)
- [TASKS_COMPLETED]: Built and configured the 6-panel frontend observability dashboard (`dashboard.html` and `config/dashboard.json`) with real-time metrics visualization, SLO compliance indicators, and auto-refresh functionality. Upgraded system from mock LLM to real OpenAI API integration (`app/openai_llm.py`) enabling authentic cost tracking, token usage, and model performance data in Langfuse. Collected comprehensive evidence including screenshots, metrics validation, and incident detection documentation.
- [EVIDENCE_LINK]: 
  - Dashboard Implementation: `dashboard.html`, `config/dashboard.json`, `serve_dashboard.py`
  - OpenAI Integration: `app/openai_llm.py`, `app/agent.py` (upgraded from mock_llm)
  - Evidence Collection: `screenshot/Dashboard 6 panel.png`, `screenshot/Dashboard langfuse.png`, `screenshot/Tracing langfuse.png`
  - Real LLM Integration: Replaced `FakeLLM` with `OpenAILLM` for authentic Langfuse model costs and usage data

### Nguyễn Bình Thành - 2A202600138 (Member F)
- [TASKS_COMPLETED]: Compiled the Blueprint report, synthesized the Root Cause Analysis (RCA) in the Incident Response section, and prepared the Demo script to showcase the system. Led the team demo presentation flow (baseline → incident injection → recovery) and explained the observability findings live. Fixed the Langfuse issue where traces were not grouped consistently by correlation ID by propagating `correlation_id` across request handling and deriving a deterministic `trace_id` for Langfuse.
- [EVIDENCE_LINK]:
  - Langfuse correlation ID tracking fix: https://github.com/hanhieu/Day13-HanQuangHieu-2A202600056/commit/097b7a0546fd5ca11be9476584ce5b9f6f17dd2d
  - Correlation ID middleware foundation: https://github.com/hanhieu/Day13-HanQuangHieu-2A202600056/commit/b86f4a4625c711f00423d582e18f12d296bfac94
  - Blueprint and demo ownership: docs/blueprint.md and demo flow in Section 4 (Incident Response)

---

## 6. Bonus Items (Optional)
- [BONUS_COST_OPTIMIZATION]: Implemented real OpenAI API integration with cost-effective model selection (gpt-3.5-turbo default, gpt-4 for cost_spike scenarios) and token limit controls (max 150 tokens per response). Added real-time cost tracking and budget alerts.
- [BONUS_AUDIT_LOGS]: Enhanced structured logging with comprehensive audit trail including correlation IDs, user context, session tracking, and PII redaction. All requests tracked with full observability chain.
- [BONUS_CUSTOM_METRIC]: Created comprehensive dashboard with 6 custom panels including Hallucination % (derived from quality scores), real-time SLO compliance indicators, and time-series visualization with 30-second auto-refresh.
