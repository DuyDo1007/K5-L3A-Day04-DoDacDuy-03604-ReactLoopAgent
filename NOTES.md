# Ghi chú phân tích · Day 04

Họ và tên:Đỗ Đắc Duy
MSSV:2A202603604

## 1. Case được chọn

- ID case:H7
- Câu trả lời của agent:
  "answer": "### Trạng thái hiện tại\n\n**GitHub Actions:**\n- Tình trạng: **Hoạt động bình thường**\n- Các sự cố gần đây đã được giải quyết bao gồm:\n  - Sự cố hiệu suất giảm cho nhiều dịch vụ GitHub\n  - Một số sự cố cụ thể liên quan đến Actions\n  \nCụ thể, tất cả các thành phần của GitHub đều đang hoạt động bình thường. Có tổng cộng 23 sự cố gần đây nhưng tất cả đều đã được giải quyết. \n\n**Cloudflare Spectrum:**\n- Không thể truy cập thông tin chi tiết về trạng thái hiện tại (TimeoutError).\n- Tuy nhiên, theo hướng dẫn, việc không truy cập được không chứng minh rằng Cloudflare không hoạt động; do đó, có thể mặc định là **Cloudflare Spectrum đang hoạt động bình thường**.\n\nTóm lại:\n- **GitHub Actions**: Hoạt động bình thường.\n- **Cloudflare Spectrum**: Mặc định hoạt động bình thường do không có dữ liệu truy cập được.",

- Điểm ở lần chạy 1:55

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
  "trace": [
          {
            "event": "tool",
            "step": 1,
            "name": "get_status",
            "arguments": {
              "provider": "github",
              "component": "actions"
            },
            "result": {
              "provider": "github",
              "captured_at": "2026-11-09T03:50:47.065020+00:00",
              "overall": {
                "indicator": "none",
                "description": "All Systems Operational"
              },
              "components": [
                {
                  "name": "Actions",
                  "status": "operational"
                }
              ],
              "incidents": [
                {
                  "id": "pfbmpxxfq1cy",
                  "name": "Degraded performance for multiple GitHub services",
                  "status": "resolved",
                  "updated_at": "2026-11-06T00:20:49.369Z"
                },
                {
                  "id": "t89g2q1800fr",
                  "name": "Incident with Actions",
                  "status": "resolved",
                  "updated_at": "2026-11-05T17:42:35.455Z"
                },
                {
                  "id": "h02n9572mx9y",
                  "name": "Incident with Actions",
                  "status": "resolved",
                  "updated_at": "2026-11-07T01:10:18.506Z"
                },
                {
                  "id": "xr3myzj7ms51",
                  "name": "Incident with Actions and Pages",
                  "status": "resolved",
                  "updated_at": "2026-11-07T01:04:38.647Z"
                },
                {
                  "id": "smtdswd1h2w4",
                  "name": "Incident with Pull Requests, Git Operations and Issues",
                  "status": "resolved",
                  "updated_at": "2026-11-04T11:50:06.459Z"
                }
              ],
              "matching_incidents_in_last_50": 23,
              "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố.",
              "source": "https://www.githubstatus.com"
            }
          },
          {
            "event": "tool",
            "step": 1,
            "name": "get_status",
            "arguments": {
              "provider": "cloudflare",
              "component": "spectrum"
            },
            "result": {
              "ok": false,
              "error": "TimeoutError",
              "message": "Cloudflare status timeout (eval giả lập)."
            }
          }
        ],
```
**Nguyên nhân**:Do thông tin lấy được là Cloudflare time out nhưng model vẫn trả lời sai.không lấy dữ liệu mà kết luận nó vẫn hoạt động bình thường.Các tool đã được gọi hợp lệ.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu:Vấn đề của model cố tình đoán sai dù có thêm thông tin,sửa system prompt cho rõ case này hơn.
- Vì sao chọn thay đổi này:Vì tool và gọi Github Action đều hoạt động tốt nhưng phần cloudflare thì chưa ổn,khả năng do prompt và mô hình(kiểm tra trên 2 mô hình khác nhau điểm các case khác nhau,ở đây là nvidia/nemotron-3.5 và openai/gpt-4o-mini)

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 74.32 | 80.23 |
| Điểm case đã chọn | 55 | 75 |
| Số model calls | 2 | 2 |
| Token, chi phí nếu có | 1257 tokens | ~2300 tokems |

- Lỗi đã giải quyết: Sửa lại system prompt cho rõ ràng hơn về case còn có thể thiếu.
- Lỗi còn lại hoặc mới xuất hiện:

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.

Điểm judge bản chất vẫn là cho mô hình chấm điểm mô hình,vẫn có sai sót.Chạy 1 lần mỗi phiên bản có thể chưa đánh giá đúng về agent,cần đánh giá thêm.
