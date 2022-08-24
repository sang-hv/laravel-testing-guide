# Feature Tests

Mục đích của Integration Test là để chắc chắn rằng tất cả các components làm việc với nhau như ta mong muốn.

Khi thực hiện kiểu test này, hãy nhớ rằng những thay đổi với data storage hay những thao tác với các service bên ngoài là có thể xảy ra.

Để có thể giải quyết một cách đúng đắn những tình huống như vậy, có thể bạn vẫn cần phải tạo các [Test Doubles](../Knowledge.md#test-doubles) cho một vài class.

Việc những Class nào cần đến Test Doubles thì phụ thuộc vào cách tiếp cận của từng team, và có thể khác biệt phụ thuộc vào độ lớn của application.

## Guide
Feature test PHẢI cover được TOÀN BỘ route của application, và phải tuân thủ các mục đích sau:

**HTTP**
- Authentication và Policy tests. Mỗi test case cần có những test assertion riêng biệt.
- Kiểm tra Status Codes cho từng loại response.
- Kiểm tra Redirect Codes và Paths cho các các sự kiện khác nhau.
- Kiểm tra tính đúng đắn của JSON responses.
- Kiểm tra Error Handlers cho các response.
- Kiểm tra tính đúng đắn của API versions (nếu cần thiết).

**Database**
- Đảm bảo data được ghi vào database một cách đúng đắn trong quá trình xử lý requests.
- Kiểm tra quá trình migration.
- Kiểm tra việc insert dữ liệu không bình thường thì có được xử lý chính xác hay không.