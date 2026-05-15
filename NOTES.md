# Ghi chú (tiếng Việt)

## Chiến lược

Dùng `serverless-http` làm adapter để chạy ứng dụng Express trên AWS Lambda.

## Tại sao chọn cách này

- Giữ nguyên `app.js` (không thay đổi logic ứng dụng).
- Chỉ cần một file entrypoint nhỏ (`lambda.js`) và thêm dependency `serverless-http`.
- Vẫn giữ được cách chạy local với `server.js` để phát triển thuận tiện.

## Cold start

- Cold start (Init Duration) sẽ đo sau khi deploy trên AWS và kiểm tra CloudWatch Logs.

## Triển khai (các bước để tái hiện)

- Yêu cầu trước: đã cấu hình `aws` CLI có quyền deploy, cài `sam` CLI, Node 18+/22.
- Các lệnh build & deploy (chạy trong thư mục dự án):

```bash
npm install
sam build
sam deploy --guided --region us-west-2
```

- Khi chạy `sam deploy --guided`, có thể dùng các giá trị khuyến nghị:
	- Stack Name: `byol-node-express`
	- Xác nhận region là `us-west-2` và chấp nhận các mặc định còn lại.

- Sau khi deploy xong, output CloudFormation `ApiUrl` chính là API Gateway URL.

API Gateway URL (đã deploy): https://7exc1p9qc8.execute-api.us-west-2.amazonaws.com

Test endpoint:
```bash
curl https://7exc1p9qc8.execute-api.us-west-2.amazonaws.com/
# Response: {"ok":true,"runtime":"express","message":"hello from your existing app"}
```

## Giữ thay đổi code ở mức tối thiểu

- Không sửa `app.js`.
- Thêm `lambda.js` dùng `serverless-http` để chuyển Express thành Lambda handler.
- Giữ `server.js` để chạy local bằng `npm start`.

## Cách đo cold start

1. Deploy stack (xem phần lệnh ở trên).
2. Đảm bảo không có container ấm (chờ >10 phút hoặc thay đổi cấu hình hàm để buộc khởi tạo mới).
3. Gọi lần đầu endpoint public (cold) bằng `curl`:

```bash
curl -sS -w "\\nHTTP:%{http_code}\\n" -o /dev/null 'https://YOUR_API_HOST/'
```

4. Kiểm tra CloudWatch Logs cho log `REPORT` chứa `Init Duration`.

Ví dụ (dùng AWS CLI):

```bash
aws logs filter-log-events --log-group-name /aws/lambda/byol-node-express --limit 20 --query "events[*].message" --output text
```

Tìm dòng chứa `Init Duration: 123.45 ms` ở lần gọi đầu (cold).

Measured cold start (Init Duration): <paste Init Duration ms here>

