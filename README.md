# Online Retail – Phân tích và dự đoán chi tiêu khách hàng

Đồ án IE313: phân tích dữ liệu giao dịch bán lẻ trực tuyến và dự đoán chi tiêu của khách hàng trong ba tháng tiếp theo bằng các mô hình hồi quy.

## Giới thiệu bộ dữ liệu

Bộ dữ liệu [Online Retail](https://archive.ics.uci.edu/dataset/352/online+retail) do UCI Machine Learning Repository cung cấp, ghi lại các giao dịch từ 01/12/2010 đến 09/12/2011 của một doanh nghiệp bán lẻ trực tuyến tại Anh. Doanh nghiệp chủ yếu bán quà tặng, nhiều khách hàng là người mua sỉ.

Bài toán là **hồi quy ở mức khách hàng**: dùng lịch sử mua hàng trước 01/06/2011 để dự đoán `FutureSpend`, tổng chi tiêu (GBP) của khách trong cửa sổ 01/06–31/08/2011. Không dự đoán `Quantity × UnitPrice` bằng chính hai biến đó, vì đó là bài toán tầm thường và rò rỉ mục tiêu.

### Tổng quan bộ dữ liệu

| Thuộc tính | Mô tả |
| --- | --- |
| Nguồn | UCI Machine Learning Repository, mã dataset `352` |
| Lĩnh vực | Bán lẻ trực tuyến, kinh doanh |
| Tác giả | Daqing Chen |
| Thời gian | 01/12/2010 đến 09/12/2011 |
| Kích thước | 541.909 dòng × 8 cột; một dòng là một dòng hàng trong hóa đơn |
| Định dạng | Excel `.xlsx`, một sheet có dòng tiêu đề |
| Đơn vị tiền | Bảng Anh (GBP) |
| Giấy phép | [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) |
| DOI | [10.24432/C5BW33](https://doi.org/10.24432/C5BW33) |
| SHA256 file gốc | `43465a06f2ccf7c8b5bd2892bc7defb52f97487934fe93b16ae4c3936424676d` |

### Từ điển dữ liệu gốc

| Nhóm | Biến | Loại | Ý nghĩa |
| --- | --- | --- | --- |
| Hóa đơn | `InvoiceNo` | Định danh | Mã hóa đơn 6 ký tự; bắt đầu bằng `C` là hóa đơn hủy |
| Sản phẩm | `StockCode` | Định danh | Mã hàng; hàng hóa có 5 chữ số, có thể kèm hậu tố chữ cái |
| Sản phẩm | `Description` | Văn bản | Tên hàng |
| Dòng hàng | `Quantity` | Số | Số lượng trên dòng hàng |
| Hóa đơn | `InvoiceDate` | Thời gian | Ngày giờ lập hóa đơn |
| Dòng hàng | `UnitPrice` | Số | Đơn giá, GBP |
| Khách hàng | `CustomerID` | Định danh | Mã khách hàng 5 chữ số (Excel lưu dạng số) |
| Khách hàng | `Country` | Phân loại | Quốc gia của khách |

### Bảng khách hàng dùng cho mô hình

Notebook `analysis/` gom dữ liệu đã làm sạch thành `dataset/NhomXX_Du_lieu_mo_hinh.csv`, mỗi dòng là một khách hàng có giao dịch trước mốc 01/06/2011.

| Biến | Vai trò | Định nghĩa | Đơn vị |
| --- | --- | --- | --- |
| `Monetary` | X | Tổng chi tiêu lịch sử | GBP |
| `Frequency` | X | Số hóa đơn khác nhau | hóa đơn |
| `Recency` | X | Số ngày từ lần mua cuối đến mốc (có phần lẻ) | ngày |
| `Tenure` | X | Số ngày từ lần mua đầu đến mốc (có phần lẻ) | ngày |
| `Products` | X | Số mã hàng khác nhau | mã hàng |
| `IsUK` | X | Quốc gia phổ biến trong lịch sử là United Kingdom | 0/1 |
| `FutureSpend` | y | Chi tiêu từ 01/06 đến trước 01/09/2011; khách không mua nhận 0 | GBP |
| `Split` | Phép chia | `train` (70%) hoặc `test` (30%), `random_state = 42` | |

### Lưu ý chất lượng dữ liệu và mô hình hóa

- **Dữ liệu khuyết:** trang UCI ghi không khuyết, nhưng file thực tế thiếu `CustomerID` ở 24,927% số dòng và thiếu `Description` ở 1.454 dòng. Không điền `CustomerID` vì đây là mã định danh.
- **Giao dịch ngoài phạm vi:** hóa đơn hủy, `Quantity`/`UnitPrice` không dương và mã phí/dịch vụ (`POST`, `M`, `C2`, ...) bị loại; mục tiêu là chi tiêu mua hàng dương, không phải doanh thu thuần.
- **Dòng trùng:** 5.268 dòng trùng toàn bộ 8 cột được loại như một giả định làm sạch, có kiểm tra độ nhạy bằng phương án giữ trùng.
- **Rò rỉ dữ liệu:** X chỉ dùng giao dịch trước mốc; y chỉ dùng giao dịch trong cửa sổ mục tiêu. EDA liên quan y, chuẩn hóa và chọn mô hình chỉ dùng tập train.
- **Phân phối lệch:** hơn một nửa số khách có y = 0 và y lệch phải mạnh; RMSE bị chi phối bởi một số khách mua sỉ, nên báo cáo thêm MAE và trung vị sai số.

## Quy trình

| Notebook | Phase | Nội dung |
| --- | --- | --- |
| `analysis/NhomXX_Phan_tich.ipynb` | 1. Thu thập và hiểu dữ liệu | Nạp dữ liệu, kiểm tra cấu trúc, kiểm kê vấn đề |
| | 2. Tiền xử lý | Lọc tuần tự có nhật ký, tạo bảng khách hàng, chia train/test |
| | 3. EDA (train) | Thống kê mô tả, mức tập trung chi tiêu, trực quan, Pearson/Spearman, ANOVA, kiểm tra giả định và độ lớn hiệu ứng |
| `predict/NhomXX_Du_doan.ipynb` | 4. Chuẩn bị dữ liệu cho mô hình | Tương quan giữa đặc trưng, VIF, Pipeline |
| | 5. Mô hình hóa | Baseline, SLR, MLR, đa thức, Ridge + GridSearchCV; chọn bằng CV 5 fold; learning curve |
| | 6. Đánh giá | Chỉ số trên test, khoảng tin cậy bootstrap, hiệu chỉnh theo decile, sai số theo nhóm, biểu đồ chẩn đoán |
| | 7. Diễn giải | Hệ số, khoảng tin cậy OLS và bootstrap cho SLR, kiểm tra độ nhạy giữ dòng trùng, demo |

## Cách chạy

Dùng Python 3.12. Cài thư viện và JupyterLab (hoặc dùng VS Code có hỗ trợ notebook):

```bash
python -m pip install -r NhomXX_Requirements.txt jupyterlab
python -m jupyter lab
```

Chạy lần lượt từ đầu đến cuối, **notebook `analysis/` trước**, sau đó `predict/`. Notebook predict kiểm tra và báo lỗi rõ ràng nếu chưa có các file do analysis tạo.

Notebook tìm dữ liệu ở thư mục hiện tại hoặc ở `../dataset`, nên chạy được theo hai cách đặt file:

- **Cấu trúc repo:** giữ nguyên các thư mục như bên dưới.
- **Nộp từng file:** đặt hai notebook và `Nhom*_Online_Retail.xlsx` chung một thư mục.

Kết quả luôn được ghi vào thư mục `Nhom*_Ket_qua/` cạnh file dữ liệu. Tiền tố nhóm lấy theo tên file dữ liệu, nên đổi `NhomXX` thành tên nhóm thực tế (ví dụ `Nhom18`) không phải sửa code. Thư mục làm việc của kernel phải là thư mục chứa notebook (mặc định của JupyterLab và VS Code).

## Cấu trúc thư mục

```text
ecomanalysis/
|-- README.md
|-- NhomXX_Requirements.txt                 # Phiên bản thư viện đã dùng để chạy
|-- analysis/
|   `-- NhomXX_Phan_tich.ipynb              # Phase 1–3: hiểu dữ liệu, tiền xử lý, EDA
|-- predict/
|   `-- NhomXX_Du_doan.ipynb                # Phase 4–7: mô hình hóa và đánh giá
`-- dataset/
    |-- NhomXX_Online_Retail.xlsx           # Dữ liệu gốc UCI, giữ nguyên byte
    |-- NhomXX_Du_lieu_mo_hinh.csv          # Bảng khách hàng (do analysis tạo)
    `-- NhomXX_Ket_qua/                     # Kết quả do hai notebook tạo (không đưa lên git)
        |-- figures/                        # 01–07 dùng trong báo cáo và slide; 08 learning curve bổ sung
        |-- cleaning_audit.csv, raw_missing.csv, descriptive_train.csv, pearson_train.csv, spearman_train.csv
        |-- customers_keep_duplicates.csv   # Bảng khách khi giữ dòng trùng (kiểm tra độ nhạy)
        |-- analysis_summary.json           # Tóm tắt Phase 1–3, đầu vào cho predict
        |-- ridge_grid.csv, model_selection.csv, learning_curve.csv
        |-- test_metrics.csv, test_predictions.csv, test_bootstrap_ci.csv, test_calibration_deciles.csv
        |-- demo_customers.csv
        `-- summary.json                    # Tóm tắt toàn bộ kết quả
```

Báo cáo, slide và hướng dẫn nội bộ được nộp riêng theo yêu cầu môn học, không lưu trong repo này.

## Tài liệu tham khảo

- **Dữ liệu:** Chen, D. (2015). *Online Retail* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5BW33.
- **scikit-learn:** [Common pitfalls and recommended practices](https://scikit-learn.org/stable/common_pitfalls.html), [GridSearchCV](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html), [Metrics and scoring](https://scikit-learn.org/stable/modules/model_evaluation.html).
- **SciPy:** [scipy.stats.f_oneway](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.f_oneway.html).
- **Bài giảng IE313:** Bài 1–7 và Kỹ thuật lập trình Python cho phân tích dữ liệu.
