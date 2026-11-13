# [Phân tích] So sánh Trade-off các giải pháp ra lệnh cho Cấu trúc Prompt Tiêu chuẩn & Định dạng
Giải pháp 1: Đơn giản, ngắn gọn (Zero-shot, chỉ định dạng cơ bản)
Trích xuất tên khách hàng, số điện thoại, sản phẩm và địa chỉ từ đoạn tin nhắn sau và trả về dưới dạng JSON:
"Chào shop, mình là Minh Anh (0901234567). Cho mình đặt 2 áo thun trắng size L giao về 123 Lê Lợi, Q1 nhé."
Giải pháp 2: Chi tiết, nhiều ràng buộc (Áp dụng CREATE + Few-shot + Ép định dạng JSON Schema)
[Role & Task] Bạn là hệ thống trích xuất dữ liệu đơn hàng tự động. Hãy phân tích tin nhắn khách hàng và xuất dữ liệu dưới dạng JSON hợp lệ.
[Constraints - Ràng buộc & Bẫy dữ liệu]
Chỉ trả về chuỗi JSON thuần túy, tuyệt đối không kèm giải thích hay ký tự markdown (như ```json).

Nếu thiếu trường thông tin nào, điền giá trị là null (không tự ý suy đoán).

Số điện thoại chuẩn hóa liền nhau (bỏ dấu cách/chấm).
[Few-shot Example]
Input: "Lấy cho anh Nam 1 đôi giày size 42, sđt 098.765.4321"
Output: {"customer_name": "Nam", "phone": "0987654321", "items": [{"product": "giày size 42", "quantity": 1}], "address": null}
[Input cần xử lý]
"Chào shop, mình là Minh Anh (0901234567). Cho mình đặt 2 áo thun trắng size L giao về 123 Lê Lợi, Q1 nhé."
## Bước 2: Lập bảng so sánh Trade-off
| Tiêu chí | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết - CREATE + Few-shot) |
| :--- | :--- | :--- |
| **Độ dễ hiểu / Viết nhanh** | **Cao** (Viết mất 10 giây, trực quan) | **Trung bình** (Cần tư duy thiết kế cấu trúc và ví dụ mẫu) |
| **Tốn token** | **Ít** (~45 tokens/lượt gọi) | **Nhiều hơn** (~180 tokens/lượt gọi, tốn chi phí API hơn) |
| **Tính nhất quán cấu trúc (Key)** | **Thấp** (Lúc dùng `ten_khach`, lúc dùng `name`) | **Rất cao** (Cố định 100% tên trường theo Few-shot) |
| **Chặn lỗi biên (Edge cases)** | **Trung bình** (Dễ bị dính text thừa *"Dưới đây là JSON..."* gây lỗi `JSON.parse()`, hoặc tự bịa địa chỉ khi thiếu) | **Cực tốt** (Xử lý chuẩn giá trị `null`, chuẩn hóa SĐT và ép output sạch để đưa thẳng vào code) |
## Bước 3: Lựa chọn và chạy thử 
1. Phương án tối ưu nhất: Giải pháp 2 (Chi tiết)
Lý do lựa chọn: Trong bài toán tích hợp hệ thống (automation/backend), tính ổn định của cấu trúc dữ liệu (Schema) và khả năng chặn lỗi biên quan trọng hơn chi phí token. Giải pháp 1 tuy rẻ và nhanh nhưng dễ làm sập hệ thống khi AI trả về thừa lời dẫn hoặc tự đổi tên key. Giải pháp 2 giải quyết triệt để vấn đề này nhờ kết hợp khung CREATE (ràng buộc đầu ra) và Few-shot (định hình cấu trúc).
2. Kết quả chạy thử (Output thực tế từ Giải pháp 2)
{"customer_name": "Minh Anh", "phone": "0901234567", "items": [{"product": "áo thun trắng size L", "quantity": 2}], "address": "123 Lê Lợi, Q1"}