# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi nhóm 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, E-seeded_regression-20261117 |
| Triệu chứng | Grader `TrialPass` (lab8) và `RefundAmountCorrect` fail: status resolved != denied; reply nói "LogiX đã hoàn 20.000đ cho đơn O-202" trong khi ledger ghi 20.000đ nhưng nhãn yêu cầu 0đ (denied) |
| Bước sai đầu tiên | Event 3: `model_override` payload: `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event 1 (`handoff`) và Event 2 (`investigator`) thực hiện truy vấn hợp lệ từ DB đơn hàng O-202 (`delay_days = 0`). Đến Event 3, agent lập tức ghi đè dữ liệu DB bằng thông tin khách hàng tự khai trong ticket (5 ngày) |
| Failure mode | FM-03 (ticket_trust — tin ticket hơn DB) |
| Đối chứng | Trên shape `fixed` (cùng C-002@11): Không có Event `model_override`. Event 3 là `quote` trả về amount_vnd = 0 ("no_entitlement"), và reply từ chối đúng theo chính sách |
| Regression test | `test_override_event_breaks_db_fact` (kiểm tra `DbFactKept().evaluate(trial)` trả về False) và `RefundAmountCorrect` (bắt lỗi chênh lệch số tiền hoàn) |
| Chưa chắc chắn | Trace chỉ ghi nhận hành vi override dữ liệu tại event 3, không chứng minh được model có bị ảo giác logic suy luận trước đó hay chỉ đơn thuần là lỗi ưu tiên trường dữ liệu |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
