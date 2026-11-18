# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi bạn 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11` |
| Triệu chứng | Grader `TrialPass` fail (`status resolved != denied`); `RefundAmountCorrect` fail (ledger ghi 20.000đ != nhãn 0đ). Reply báo đã hoàn 20.000đ trong khi đơn hàng chỉ trễ 2 ngày và phải bị từ chối. |
| Bước sai đầu tiên | Event #3 `model_override` `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event #1 (`handoff`) đúng. Event #2 (`investigator`) tra cứu DB trả về dữ liệu chuẩn xác `order.delay_days = 2`. Event #3 ngay sau đó đã lấy số 5 từ ticket khách hàng đè lên DB, dẫn đến tính toán quote sai ở Event #4 và hoàn tiền sai. |
| Failure mode | `FM-03` (ticket_trust — tin ticket hơn DB) |
| Đối chứng | Cùng `C-002@11` trên shape `fixed`: sau Event #2 chuyển thẳng sang Event #3 `quote` `{"amount_vnd": 0, "reason": "no_entitlement"}` (không có event `model_override`), ledger ghi 0đ, reply từ chối đúng quy định. |
| Regression test | `test_override_event_breaks_db_fact` (với grader `DbFactKept`) và `test_public_single_run_cases[swarm-C-002-11]`. Đã quan sát thấy FAIL trên `seeded_regression` và PASS trên `fixed`. |
| Chưa chắc chắn | Trace chỉ ghi nhận hành vi ghi đè dữ liệu, không thể hiện được prompt reasoning nội tại của model vì sao lại chọn tin ticket hơn DB. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
