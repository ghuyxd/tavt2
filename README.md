# Bộ chuyển đổi Latin → chữ Thái Việt Nam (Tai Viet, U+AA80–U+AADF) v2

Dự án này cung cấp công cụ chuyển đổi văn bản từ chữ Latin sang chữ Thái Việt Nam. Mô hình được huấn luyện trực tiếp từ dữ liệu thực tế kết hợp với các quy tắc ngôn ngữ, mang lại độ chính xác cao trong việc chuyển ngữ.

## 🚀 Cài đặt và Sử dụng

```bash
# Huấn luyện mô hình từ dữ liệu từ điển và xuất ra file model.json
python3 build_model.py

# Đánh giá lại mô hình trên tập từ điển
python3 tavt2.py --eval

# Chuyển đổi một văn bản thông thường
python3 tavt2.py vanban.txt -o ketqua.txt -r baocao.txt

# Chuyển đổi dữ liệu từ file CSV (chỉ định cột cần chuyển)
python3 tavt2.py --csv tudien.csv --col "Từ Thái (Latinh)" -o ketqua.csv
```

> **Lưu ý về nguồn dữ liệu:**
> Dự án hỗ trợ hai quy ước chính tả khác nhau. Vui lòng chọn cờ `-p` tương ứng với nguồn dữ liệu để đạt kết quả tốt nhất:
> ```bash
> python3 tavt2.py vanban.txt -p textbook -o ketqua.txt   # Theo giáo trình Tản Chụ Xiết Xương
> python3 tavt2.py vanban.txt -p dict     -o ketqua.txt   # Theo từ điển Sơn La (mặc định)
> ```

## 📊 Hiệu suất và Đánh giá

Hệ thống đạt được các chỉ số ấn tượng sau khi đánh giá trên tập dữ liệu:

| Tiêu chí | Độ chính xác | Ý nghĩa sử dụng |
|---|---|---|
| **[A] Mục từ có trong từ điển** | **99,91%** (1072/1073) | Tra bảng — Đảm bảo chính xác cho toàn bộ vốn từ đã có |
| **[B] Âm tiết đã gặp, dùng quy tắc** | 95,83% | Phân tích và dự đoán không cần tra bảng |
| **[C] Âm tiết mới (Quy tắc, 5-fold CV)**| 93,36% | Dự đoán độc lập từng âm tiết mới |
| **[D] Mục từ hoàn toàn mới (5-fold CV)** | 61,42% | Yêu cầu khớp toàn bộ cụm 2–3 âm tiết |

Mô hình đảm bảo độ chính xác gần như tuyệt đối (>99.9%) cho các từ đã có trong từ điển. Tuy nhiên, đối với các từ mới hoàn toàn, giới hạn vật lý của chữ Latin (không ghi đủ thông tin thanh điệu) khiến mức trần dự đoán rơi vào khoảng ~93% cho mỗi âm tiết.

## 🧠 Kiến trúc và Xử lý Dữ liệu

### 1. Vấn đề thiếu thông tin thanh điệu
Chữ Thái Việt Nam mã hoá thanh điệu thông qua **lớp phụ âm đầu** (mỗi phụ âm có 2 dạng LOW/HIGH). Tuy nhiên, nguồn từ điển sử dụng hệ truyền thống không đánh dấu thanh. Do đó, mô hình loại bỏ dấu thanh và sử dụng lớp phụ âm để mang thanh.

Hệ thống học ánh xạ thanh điệu tiếng Việt sang lớp phụ âm từ dữ liệu, kèm theo độ nhất quán cao (hỏi → LOW: 99,6%, nặng → HIGH: 99,6%...). Trong trường hợp dữ liệu thiếu, mô hình sẽ sử dụng chung quy tắc `(thanh)` kết hợp `(thanh, phụ âm đầu)` để cải thiện độ chính xác.

### 2. Tiền xử lý và Làm sạch Dữ liệu
Dữ liệu đầu vào trải qua ba tầng xử lý nghiêm ngặt:
- **Loại bỏ mâu thuẫn:** Loại bỏ các cặp từ có phụ âm đầu hoặc mẫu vần trái ngược với ánh xạ đa số.
- **Sửa lỗi định dạng (repair.py):** Khắc phục lỗi cắt chữ do OCR hoặc lỗi tách rời dấu phụ Unicode, giúp nâng hiệu suất tra bảng lên 99,9%.
- **Lọc dữ liệu:** Loại bỏ các nhiễu OCR thô để đảm bảo chất lượng cho corpus huấn luyện.

## 📁 Cấu trúc thư mục

| Tập tin / Thư mục | Mô tả nội dung |
|---|---|
| `model.json` | Bảng ánh xạ phụ âm, vần, thanh điệu cùng từ điển 729 âm tiết & 1.071 mục từ. |
| `tavt2.py` | Engine chuyển đổi chính với quy trình 3 tầng (từ → âm tiết → quy tắc). |
| `build_model.py` | Script huấn luyện và đánh giá mô hình từ file CSV gốc. |
| `repair.py` | Script hỗ trợ sửa lỗi tách từ trong dữ liệu nguồn. |
| `tables_textbook.py`| Bảng chuẩn hóa số liệu theo giáo trình Tản Chụ Xiết Xương. |
| `taiviet_dictionary_clean.csv` | Corpus đã qua làm sạch gồm 1.109 mục, sẵn sàng để huấn luyện. |
| `errors.txt` | Nhật ký lỗi chi tiết hỗ trợ đối chiếu thủ công. |

## 🔮 Hướng phát triển

Để vượt qua mức trần 93% hiện tại trên tập từ mới, dự án cần tập trung vào:
1. **Mở rộng từ điển:** Việc bổ sung thêm ~600 âm tiết bằng phương pháp thủ công sẽ giúp bao phủ phần lớn các văn bản thực tế.
2. **Cải thiện OCR:** Giữ nguyên sự phân biệt chữ hoa/thường ở cột chữ Việt trong các tài liệu gốc (như giáo trình) để mã hóa chính xác lớp thanh mà không cần phải dự đoán.

## 🌟 Credits

Thuật toán nền tảng của chương trình này thực chất là thuật toán đảo ngược dựa trên bộ quy tắc chuyển đổi chữ Thái Việt Nam từ dự án Unicode CLDR:
- Tham khảo bản gốc: [unicode-org/cldr: blt-fonipa-t-blt.xml](https://github.com/unicode-org/cldr/blob/main/common/transforms/blt-fonipa-t-blt.xml)
