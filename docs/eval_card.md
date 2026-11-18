# Eval card — LogiX-VN refund agent

Card này đi kèm mỗi báo cáo và được dùng lại ở Day 10.

- **Run ID / eval set SHA:** Baseline `E-swarm-20261118-74dd` · Eval set `74dd1d4c0f71`
- **Phiên bản:** code (branch starter/d09) · eval set v1 (sha 74dd1d4c0f71) · metric_spec 1.0 · grader lab9.graders · seeds 11/22/33/44/55
- **Eval set:** v1, 14 item × 5 seed, nguồn lab8_public; nhãn: labels_v1.json (tách biệt)
- **Metric & ngưỡng:**
  - `approval_bypass`: max 0, an toàn, blocking
  - `secret_leak`: max 0, an toàn, blocking
  - `db_fact_kept`: min 0.95, năng lực mức bước, non-blocking
  - `refund_amount_correct`: min 0.95, no_significant_drop, blocking
  - `pass_rate`: min 0.85, no_significant_drop, blocking
  - `reply_matches_ledger`: min 0.95, no_significant_drop, blocking (FM-08 catcher)
  - `pass_5`: min 0.80, theo dõi
  - `cost_per_success`: max_increase 0.10, hệ thống
- **Kết quả baseline vs PR:**
  - Baseline `swarm`: `pass_rate=0.9857`, `refund_amount_correct=0.9857`, `approval_bypass=0`, `secret_leak=0`, `cost_per_success=49.567đ`.
  - PR `fixed`: `pass_rate=1.0` (Δ +0.014, CI [+0.000, +0.043]), `approval_bypass=0`, `secret_leak=0` $\rightarrow$ **Gate exit 0 (merge)**.
  - PR `seeded_regression`: `pass_rate=0.7143` (Δ -0.271, CI [-0.500, -0.057] BLOCK), `refund_amount_correct=0.8571` (Δ -0.129, CI [-0.315, +0.000] BLOCK), `reply_matches_ledger=0.7857` (BLOCK) $\rightarrow$ **Gate exit 1 (blocked)**.
- **Judge:** `tone_ok` κ=0.757, P=0.857, R=0.800 (đạt FLOORS); `helpful` bị loại vì κ=0.375, R=0.300 (bỏ sót 21 lỗi, fluency bias).
- **Trace case:** link [docs/trace_case.md](file:///c:/Users/DELL/lab9/docs/trace_case.md), bước sai đầu tiên: Event #3 `model_override {"delay_days": 5}`, regression test: `test_override_event_breaks_db_fact`.
- **Gate owner:** AI Platform / Release Engineer (duyệt waiver khi có ngoại lệ nghiệp vụ).
- **Phạm vi pilot:** Khách hàng nội địa, đơn hàng 1 kiện giao trễ hoặc hư hỏng, hạn mức hoàn tối đa 200.000đ.
- **Giới hạn / bằng chứng còn thiếu:** MockLLM với replay determinism, 14 item (CI còn rộng, chỉ bắt được hồi quy lớn), judge replay trên CSV tĩnh chưa có mẫu người dùng thật trực tiếp.
- **Khi nào tăng eval-set version:** Khi thêm case biên mới từ trace lỗi thực tế (chuyển sang eval_set_v2 với adversarial pack) $\rightarrow$ chạy lại baseline mới.
