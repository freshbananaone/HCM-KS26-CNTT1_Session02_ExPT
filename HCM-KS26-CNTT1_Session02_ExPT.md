# Bài: So sánh Trade-off các giải pháp ra lệnh cho Cấu trúc Prompt Tiêu chuẩn & Định dạng

## Bước 1: Đề xuất 2 prompt mẫu

**Bối cảnh:** Cửa hàng MartX muốn dùng AI tự động phân loại phản hồi khách hàng để nhân viên xử lý nhanh hơn.

### Giải pháp 1: Prompt đơn giản, ngắn gọn

```text
Bạn là nhân viên chăm sóc khách hàng của MartX.
Hãy phân loại các phản hồi thành 3 nhóm: TÍCH_CỰC, TIÊU_CỰC, CẦN_HỖ_TRỢ.
Trả kết quả bằng bảng Markdown gồm: Mã khách hàng | Nội dung | Phân loại.

Dữ liệu:
- KH01: Giao nhanh, rau tươi ngon.
- KH02: Rau dập nát, tôi không hài lòng.
- KH03: Thiếu 2 sản phẩm, nhờ cửa hàng kiểm tra.
- KH04: (để trống)
- KH05: 12345
```

**Ưu điểm:** Dễ viết, dễ hiểu, ít tốn token. **Hạn chế:** Chưa quy định rõ cách xử lý dữ liệu trống, sai kiểu hoặc phản hồi có nhiều ý.

### Giải pháp 2: Prompt chi tiết theo CREATE + Few-shot + JSON/Table

```text
[C - Context / Ngữ cảnh]
Bạn là trợ lý AI hỗ trợ bộ phận CSKH cửa hàng MartX, chuyên phân loại phản hồi khách hàng.

[R - Request / Yêu cầu]
Phân loại từng phản hồi vào đúng 1 trong 4 nhóm:
TÍCH_CỰC, TIÊU_CỰC, CẦN_HỖ_TRỢ, KHÔNG_HỢP_LỆ.

[E - Explanation / Giải thích]
Kết quả được dùng để đưa vào hệ thống chăm sóc khách hàng. Vì vậy cần đúng cấu trúc và không tự đoán khi dữ liệu bị thiếu hoặc sai định dạng.

[A - Action / Hành động]
1. Đọc lần lượt từng bản ghi.
2. Kiểm tra trường ma_kh và noi_dung.
3. Nếu noi_dung rỗng hoặc không phải chuỗi văn bản, gán KHÔNG_HỢP_LỆ.
4. Nếu có yêu cầu hỗ trợ, đổi/trả, kiểm tra đơn hàng thì gán CẦN_HỖ_TRỢ, kể cả khi có lời phàn nàn.
5. Nếu chỉ thể hiện không hài lòng thì gán TIÊU_CỰC; nếu chỉ khen thì gán TÍCH_CỰC.
6. Nếu nội dung là chuỗi nhưng không đủ rõ để phân loại, gán KHÔNG_HỢP_LỆ, ghi lý do KHÔNG_RÕ.
7. Trả về trước tiên một mảng JSON hợp lệ; sau đó là bảng Markdown có cùng dữ liệu.
   Các trường trong JSON: ma_kh, noi_dung, nhom, loi.
   Giá trị loi là KHÔNG, TRỐNG, SAI_KIỂU hoặc KHÔNG_RÕ.

[T - Tone / Giọng điệu]
Ngắn gọn, rõ ràng, trung lập, phù hợp với nhân viên CSKH.

[E - Extras / Ràng buộc]
- Không tự bịa nội dung hay bỏ sót bản ghi.
- Không sửa mã khách hàng và không tự chuyển số thành lời nhận xét.
- Không tạo thêm nhãn ngoài 4 nhãn quy định.
- Nếu trường ma_kh bị thiếu, dùng chuỗi "CHƯA_CÓ_MÃ" và ghi lỗi tương ứng theo quy định vận hành; không tự tạo mã thật.
- Kết quả JSON và bảng phải thống nhất.

[Few-shot: Ví dụ mẫu]
Đầu vào: {"ma_kh":"VD01","noi_dung":"Hàng tươi và giao rất nhanh"}
Đầu ra: {"ma_kh":"VD01","noi_dung":"Hàng tươi và giao rất nhanh","nhom":"TÍCH_CỰC","loi":"KHÔNG"}

Đầu vào: {"ma_kh":"VD02","noi_dung":"Rau hỏng, nhờ cửa hàng đổi giúp"}
Đầu ra: {"ma_kh":"VD02","noi_dung":"Rau hỏng, nhờ cửa hàng đổi giúp","nhom":"CẦN_HỖ_TRỢ","loi":"KHÔNG"}

Đầu vào: {"ma_kh":"VD03","noi_dung":""}
Đầu ra: {"ma_kh":"VD03","noi_dung":"","nhom":"KHÔNG_HỢP_LỆ","loi":"TRỐNG"}

[Dữ liệu cần xử lý]
[
  {"ma_kh":"KH01","noi_dung":"Giao nhanh, rau tươi ngon."},
  {"ma_kh":"KH02","noi_dung":"Rau dập nát, tôi không hài lòng."},
  {"ma_kh":"KH03","noi_dung":"Thiếu 2 sản phẩm, nhờ cửa hàng kiểm tra."},
  {"ma_kh":"KH04","noi_dung":""},
  {"ma_kh":"KH05","noi_dung":12345}
]
```

**Ưu điểm:** Kiểm soát định dạng và trường hợp lỗi tốt hơn, có ví dụ để AI làm theo. **Hạn chế:** Prompt dài, tốn nhiều token hơn và cần cập nhật khi quy tắc thay đổi.

## Bước 2: Bảng so sánh Trade-off

| Tiêu chí | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) |
|---|---|---|
| Độ dễ hiểu | Cao, đọc là làm được | Trung bình, cần đọc kỹ quy tắc |
| Tốn token | Ít | Nhiều hơn |
| Thời gian soạn prompt | Nhanh | Lâu hơn |
| Định dạng đầu ra | Bảng Markdown | JSON và bảng Markdown |
| Few-shot (ví dụ mẫu) | Không có | Có 3 ví dụ |
| Xử lý dữ liệu trống, sai kiểu | Chưa rõ ràng | Có điều kiện cụ thể |
| Hạn chế AI tự suy đoán | Trung bình | Tốt hơn, nhưng vẫn cần kiểm tra |
| Phù hợp | Thử nhanh, dữ liệu đơn giản | Tự động hóa với dữ liệu thực tế |

## Bước 3: Lựa chọn và chạy thử

**Em chọn Giải pháp 2** vì MartX cần đưa dữ liệu vào hệ thống. Prompt có khung CREATE, ví dụ Few-shot và ràng buộc lỗi nên kết quả dễ kiểm tra, đồng nhất hơn. Dù tốn nhiều token, giải pháp này phù hợp hơn khi xử lý nhiều phản hồi.

### Dữ liệu chạy thử

5 phản hồi của KH01 đến KH05 đã được đặt trong prompt ở Bước 1, trong đó KH04 bị trống và KH05 sai kiểu dữ liệu.

### Kết quả AI theo Prompt 2

**JSON:**

```json
[
  {"ma_kh":"KH01","noi_dung":"Giao nhanh, rau tươi ngon.","nhom":"TÍCH_CỰC","loi":"KHÔNG"},
  {"ma_kh":"KH02","noi_dung":"Rau dập nát, tôi không hài lòng.","nhom":"TIÊU_CỰC","loi":"KHÔNG"},
  {"ma_kh":"KH03","noi_dung":"Thiếu 2 sản phẩm, nhờ cửa hàng kiểm tra.","nhom":"CẦN_HỖ_TRỢ","loi":"KHÔNG"},
  {"ma_kh":"KH04","noi_dung":"","nhom":"KHÔNG_HỢP_LỆ","loi":"TRỐNG"},
  {"ma_kh":"KH05","noi_dung":12345,"nhom":"KHÔNG_HỢP_LỆ","loi":"SAI_KIỂU"}
]
```
