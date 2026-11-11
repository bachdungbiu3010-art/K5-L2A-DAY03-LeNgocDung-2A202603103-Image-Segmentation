# Báo cáo thực hành Day 3 — điền trực tiếp trong fork của bạn

- Mã học viên theo lớp: 2A202603103
- Ngày / CVAT local: 11/11/2026 / CVAT online (app.cvat.ai)
- Công cụ đã dùng: CVAT online; script Python tự viết (`tools/prelabel/`) để gán nháp: YOLO11x-seg (instance, pretrained COCO) và BiSeNet-M (semantic, pretrained Cityscapes / COCO-Stuff), đổi ID lớp theo `classes.json` của từng task, áp bộ quy tắc hậu xử lý R0–R5 (mô tả trong `tools/prelabel/QUY_TAC_HAU_XU_LY.md`). Bản nháp được nạp vào CVAT bằng Upload annotations rồi kiểm trên CVAT. Không dùng SAM.

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | chưa có | 0 / 1 | 3 |
| cp2_slice | chưa có | 0 / 1 | 3 |
| cp5_occlusion | chưa có | 0 / 1 | 3 |
| cp3_thin | chưa có | 0 / 1 | 3 |
| cp4_curb | chưa có | 0 / 1 | 3 |
| cp6_coverage | chưa có | 0 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Sáu checkpoint chưa kịp export; đã thử nạp nháp cp2_slice vào CVAT (14 object) nhưng chưa sửa và export.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: ảnh `000000181542.jpg`, góc dưới bên trái — bánh trước của một xe máy bị khung ảnh cắt, chỉ nhìn thấy khoảng 1/4 bánh. Vẽ bằng Polygon trước khi nạp bất kỳ gợi ý nào (có ảnh chụp màn hình).
- Class và quy tắc tôi dùng để chọn biên: `motorcycle`. Biên bám mép ngoài lốp; nan hoa bên trong giữ trong mask (không khoét lỗ); cạnh trái và cạnh dưới dừng ở mép ảnh vì xe bị khung ảnh cắt — chỉ vẽ phần nhìn thấy, không đoán phần ngoài ảnh.
- Nếu dùng gợi ý sau đó: model cũng nhận ra chiếc xe này (`motorcycle`) và gộp 2 mảng rời — bánh xe và mảng trắng phía trên bánh — thành 1 object; bản tự vẽ của tôi chỉ có bánh xe. Bản nộp đang giữ đề xuất của model (1 object, 2 mảng) theo quy tắc "các phần nhìn thấy của cùng một vật là một instance"; mảng trắng nhiều khả năng là chắn bùn/thân của cùng xe nhưng ảnh bị cắt nên chưa chắc chắn — xem ca 2 ở mục 4.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: easy_semantic / `7ee6d192-89e2408b.jpg` / bên trái ảnh, vùng chân đồi cỏ khô và dải tường bê tông phân cách.
- Lỗi thuộc loại: sai lớp / phủ vùng.
- Bằng chứng tôi nhìn thấy: viền vùng `road` trong CVAT đi lên tận chân đồi cỏ, bao luôn tường bê tông; model chắc ~0,97 rằng tường là road và dải tối dưới chân đồi có màu giống nhựa đường.
- Quy tắc và hành động sửa: `road` chỉ là mặt đường xe chạy; ranh trên của road phải dừng ở chân tường; đồi để trống vì task không có lớp terrain. Tôi đã sửa thuật toán gán nháp (quy tắc R3c: chỉ loang mặt đường từ khối nhựa đường cùng màu nối xuống sát camera) — bản nháp mới không còn road trên đồi/tường.
- Sau sửa đã Save và export lại chưa? Chưa: file `easy_semantic.zip` đang nộp vẫn còn vùng road lấn lên chân đồi/tường ở ảnh này (bản export trước khi áp quy tắc mới). Cần coach lưu ý vùng này khi chấm.

Chưa có điểm tự đánh giá (chưa chạy scorer với reference).

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| easy `7ee6d192`, đồi cỏ khô hai bên đường | `vegetation` hay để trống | Task chỉ có road/sidewalk/building/vegetation/sky, không có terrain; cỏ khô không phải cây | Hỏi coach: đồi cỏ khô nên để trống hay tính vegetation? |
| medium `000000181542`, mảng trắng phía trên bánh xe máy góc dưới trái | Thuộc cùng xe máy (chắn bùn) hay vật khác | Các phần nhìn thấy của cùng vật là 1 instance; ảnh bị cắt nên không thấy chỗ nối | Giữ chung 1 object `motorcycle`; hỏi coach nếu phải tách |
| hard `000000460147`, xe tải chở ô tô ở giữa đường | 1 `truck` hay `truck` + các `car` trên xe tải | Model đề xuất cả truck lẫn car chồng lên nhau | Hỏi coach: ô tô chở trên xe tải có tính là instance `car` riêng không? |
