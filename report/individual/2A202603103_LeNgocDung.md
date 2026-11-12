# BÁO CÁO THỰC HÀNH DAY 3
## IMAGE SEGMENTATION – CVAT & AI PRE-LABELING

**Mã học viên:** 2A202603103  
**Ngày thực hành:** 11/11/2026  
**Môi trường:** CVAT Online (app.cvat.ai)

---

## 1. Tổng quan thực hành

### 1.1. Công cụ sử dụng

- **CVAT Online:** Gán nhãn, kiểm tra và xuất annotation.
- **Python (`tools/prelabel/`):** Xây dựng công cụ gán nhãn nháp tự động.
- **YOLO11x-seg:** Instance Segmentation, pretrained COCO.
- **BiSeNet-M:** Semantic Segmentation, pretrained Cityscapes / COCO-Stuff.
- **`classes.json`:** Ánh xạ ID lớp theo từng Task.
- **Quy tắc hậu xử lý R0–R5:** Mô tả tại `tools/prelabel/QUY_TAC_HAU_XU_LY.md`.

**Quy trình đã thực hiện:**

1. Sử dụng mô hình AI để tạo nhãn gợi ý.
2. Ánh xạ ID lớp theo `classes.json` của từng Task.
3. Áp dụng các quy tắc hậu xử lý R0–R5.
4. Với 3 Task chính, nạp annotation vào CVAT bằng chức năng `Upload annotations`.
5. Xem lại kết quả trên CVAT.
6. Với 6 checkpoint, sử dụng trực tiếp bản gợi ý của công cụ.

**Lưu ý:** Không sử dụng SAM.

---

## 2. Kết quả các bài đã nộp

| Bài thực hành | File ZIP | Số ảnh | Điểm tối đa |
|---|---|---:|---:|
| easy_semantic | easy_semantic.zip | 3/3 | 20 |
| medium_instance | medium_instance.zip | 3/3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2/2 | 30 |
| cp1_holes | cp1_holes.zip | 1/1 | 3 |
| cp2_slice | cp2_slice.zip | 1/1 | 3 |
| cp3_thin | cp3_thin.zip | 1/1 | 3 |
| cp4_curb | cp4_curb.zip | 1/1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1/1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1/1 | 3 |
| **Tổng cộng** | | **14/14** | **100** |

*Ghi chú: Điểm trong bảng là điểm tối đa theo cơ cấu bài thực hành, không phải điểm Coach đã chấm.*

### 2.1. Minh bạch nguồn gốc kết quả

**Nhóm 1 – Sáu checkpoint**

Bao gồm:

`cp1_holes`, `cp2_slice`, `cp3_thin`, `cp4_curb`, `cp5_occlusion`, `cp6_coverage`.

- Là bản gợi ý tự động từ công cụ `tools/prelabel/`.
- Sử dụng định dạng COCO 1.0 / Segmentation mask 1.1.
- Không thực hiện bước export từ CVAT.

**Nhóm 2 – Medium và Hard**

- `medium_instance` và `hard_panoptic` được export từ CVAT sau khi nạp nhãn gợi ý.
- Bản export giữ nguyên đề xuất của mô hình.
- Chưa thay đổi object nào so với bản gợi ý.

**Nhóm 3 – Easy Semantic**

- `easy_semantic` được export từ CVAT.
- Kết quả khác bản gợi ý ban đầu khoảng 1% pixel.

**Nguyên tắc thực hiện:** Theo `RULES.md`, mọi đề xuất của AI cần được kiểm tra trước khi chấp nhận. Tôi ghi rõ nguồn gốc từng kết quả để Coach phân biệt phần do mô hình đề xuất và phần do học viên thực hiện.

---

## 3. Một quyết định trước khi sử dụng AI gợi ý

### 3.1. Đối tượng tự gán nhãn

| Nội dung | Mô tả |
|---|---|
| Task | medium_instance |
| Ảnh | `000000181542.jpg` |
| Vị trí | Góc dưới bên trái |
| Đối tượng | Bánh trước xe máy |
| Class | `motorcycle` |
| Công cụ | Polygon |

**Quan sát thực tế:**

Bánh trước xe máy bị mép ảnh cắt, chỉ nhìn thấy khoảng 1/4 bánh.

Tôi đã tự vẽ Polygon trước khi nạp bất kỳ gợi ý AI nào và có ảnh chụp màn hình làm minh chứng.

### 3.2. Quy tắc xác định biên

- Bám theo mép ngoài của lốp xe.
- Phần nan hoa bên trong được giữ trong mask, không khoét lỗ.
- Cạnh trái và cạnh dưới dừng tại mép ảnh.
- Chỉ gán phần nhìn thấy, không suy đoán phần nằm ngoài ảnh.

### 3.3. So sánh với AI

Sau khi sử dụng gợi ý:

- Mô hình cũng nhận diện đối tượng là `motorcycle`.
- Mô hình gộp hai vùng rời nhau gồm bánh xe và mảng trắng phía trên bánh thành một object.
- Bản tự vẽ ban đầu chỉ bao gồm bánh xe.

**Quyết định:** Bản nộp hiện giữ đề xuất của mô hình: một object gồm hai mảng.

Lý do: Các phần nhìn thấy của cùng một vật được xem là một instance.

Tuy nhiên, mảng trắng có khả năng là chắn bùn hoặc thân xe, nhưng chưa thể xác nhận chắc chắn do ảnh bị cắt. Vấn đề được ghi nhận để hỏi Coach.

---

## 4. Một lỗi đã phát hiện và sửa

### 4.1. Thông tin lỗi

| Nội dung | Chi tiết |
|---|---|
| Task | easy_semantic |
| Ảnh | `7ee6d192-89e2408b.jpg` |
| Vị trí | Bên trái ảnh, khu vực chân đồi và tường bê tông |
| Loại lỗi | Sai lớp và phủ vùng |

### 4.2. Hiện tượng

Khi kiểm tra trên CVAT, tôi phát hiện:

- Vùng `road` bị kéo lên đến chân đồi cỏ khô.
- Một phần tường bê tông bị nhận diện thành mặt đường.
- Mô hình đưa ra độ tin cậy khoảng 0,97 cho nhận định tường là `road`.
- Dải tối dưới chân đồi có màu sắc tương tự mặt nhựa đường.

### 4.3. Nguyên nhân và hướng sửa

**Quy tắc xác định:** `road` chỉ đại diện cho mặt đường xe chạy.

Vì vậy:

- Ranh trên của `road` phải dừng tại chân tường.
- Đồi cỏ khô được để trống do Task không có lớp `terrain`.
- Không mở rộng mask chỉ dựa trên sự tương đồng màu sắc.

**Hành động khắc phục:**

Tôi điều chỉnh thuật toán gán nhãn nháp theo quy tắc `R3c`:

Chỉ cho phép vùng mặt đường phát triển từ khối nhựa đường cùng màu, có tính liên thông với khu vực gần camera.

**Kết quả:** Bản gợi ý mới không còn phủ nhãn `road` lên khu vực đồi và tường.

---

## 5. Ba trường hợp cần kiểm tra hoặc xác nhận

### Ca 1 – Phân loại đồi cỏ khô

**Task:** easy_semantic  
**Ảnh:** `7ee6d192-89e2408b.jpg`

**Vấn đề:**

Đồi cỏ khô nằm hai bên đường có nên được phân loại là `vegetation` hay để trống?

**Hai khả năng:**

- Gán `vegetation` vì khu vực có thảm thực vật.
- Để trống vì Task không có lớp `terrain`, trong khi cỏ khô chưa được xác định rõ theo quy ước lớp.

**Xử lý hiện tại:** Để trống.

**Câu hỏi Coach:** Đồi cỏ khô nên để trống hay được tính vào lớp `vegetation`?

### Ca 2 – Các bộ phận rời của xe máy

**Task:** medium_instance  
**Ảnh:** `000000181542.jpg`

**Vấn đề:**

Mảng trắng phía trên bánh xe máy là bộ phận của cùng chiếc xe hay một đối tượng khác?

**Hai khả năng:**

- Thuộc cùng chiếc xe máy (chắn bùn hoặc thân xe).
- Không đủ bằng chứng để xác định mối liên hệ.

**Xử lý hiện tại:** Giữ một object `motorcycle`, gồm hai vùng rời nhau theo gợi ý AI.

**Câu hỏi Coach:** Trường hợp các bộ phận bị cắt khỏi mép ảnh và không nhìn thấy điểm nối, có được gộp chung thành một instance không?

### Ca 3 – Xe tải đang chở ô tô

**Task:** hard_panoptic  
**Ảnh:** `000000460147.jpg`

**Vấn đề:**

Xe tải chở ô tô ở giữa đường cần được gán thành một object `truck` hay nhiều object gồm `truck` và những chiếc `car` đang được vận chuyển?

**Đề xuất của mô hình:** Có cả `truck` và `car`, với các vùng annotation chồng lấn.

**Trạng thái:** Chưa có đủ thông tin trong phần báo cáo hiện tại để xác nhận quyết định cuối cùng.

**Câu hỏi Coach:** Các xe ô tô được vận chuyển trên xe tải có cần gán thành những instance riêng biệt không? Nếu có, xử lý vùng chồng lấn với xe tải như thế nào?

---

## 6. Tổng kết bài học

Qua Day 3, tôi rút ra ba nhận thức:

**1. AI giúp tăng tốc nhưng không thay thế bước kiểm tra dữ liệu.**

Mô hình có thể đưa ra dự đoán rất tự tin nhưng vẫn sai, đặc biệt khi các vùng có màu sắc hoặc đặc điểm thị giác gần giống nhau.

**2. Gán nhãn cần dựa trên quy tắc thống nhất.**

Một đối tượng có thể được hiểu theo nhiều cách nếu không xác định rõ ranh giới, class và quy tắc xử lý các phần bị che khuất hoặc bị cắt.

**3. Cần
