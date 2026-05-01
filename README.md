# Báo cáo Giải pháp Datathon - Đội NEU-Solution
# DISCLAIMER

Báo cáo phân tích và mô hình dự báo của đội **NEU-Solution** được xây dựng dựa trên tập dữ liệu do Ban tổ chức cung cấp trong khuôn khổ cuộc thi **Datathon 2026**. Để đảm bảo tính khách quan cho kết quả phân tích, chúng tôi xin lưu ý các điểm sau:

*   **Bản chất dữ liệu:** Do đặc thù của quá trình giả lập dữ liệu (*Synthetic Data*), một số chỉ số kinh doanh thực tế như giá cả (`price`), tỷ lệ thoát (`bounce_rate`), nguồn lưu lượng (`traffic_source`),... và các hành vi khách hàng có thể tồn tại những sai lệch nhất định hoặc không hoàn toàn sát với thực tế thị trường[cite: 1].
*   **Phạm vi phân tích:** Do sự thiếu tương đồng của các biến số này so với logic vận hành thực tế, đội quyết định tập trung tối đa vào việc xây dựng phương pháp luận, tối ưu hóa quy trình xử lý dữ liệu và kiểm chứng tư duy mô hình hóa thay vì đi sâu vào phân tích ý nghĩa tuyệt đối của các con số kể trên.

Chúng tôi tin rằng việc tập trung vào `methodology` sẽ mang lại giá trị phân tích bền vững và phản ánh đúng năng lực xử lý dữ liệu của nhóm trong dự án này

# Báo cáo Giải pháp Datathon - Đội NEU-Solution
Repository này lưu trữ toàn bộ mã nguồn, quy trình phân tích dữ liệu (EDA) và mô hình dự báo chuỗi thời gian (Time-series Forecasting) của đội NEU-Solution dành cho bài toán quản trị và dự báo doanh thu của một doanh nghiệp thời trang thương mại điện tử.

## 1. Kiến trúc Dữ liệu

Dự án sử dụng hàm `load_datathon_data` để tự động hóa việc tải và ép kiểu thời gian (datetime) cho 13 bảng dữ liệu thuộc 4 nhóm chính:

*   **Dữ liệu Master:** `products.csv`, `customers.csv`, `promotions.csv`, `geography.csv`.
*   **Dữ liệu Transaction (Giao dịch):** `orders.csv`, `order_items.csv`, `payments.csv`, `shipments.csv`, `returns.csv`, `reviews.csv`.
*   **Dữ liệu Analytical (Phân tích):** `sales.csv` (dữ liệu huấn luyện 2012-2022) và `sales_test.csv`.
*   **Dữ liệu Operational (Vận hành):** `inventory.csv`, `web_traffic.csv`.

---

## 2. Các Phát hiện Kinh doanh Cốt lõi (Khám phá Dữ liệu)

Quá trình truy vấn dữ liệu đã mang lại những góc nhìn định lượng quan trọng về hiện trạng doanh nghiệp:

*   **Hành vi mua sắm:** Khoảng cách trung vị giữa hai lần mua liên tiếp của khách hàng là 144 ngày. Nhóm tuổi mang lại số đơn hàng trung bình cao nhất là nhóm 55+ (5.4 đơn/khách).
*   **Hiệu suất sản phẩm:** Phân khúc Standard đem lại tỷ suất lợi nhuận gộp cao nhất, đạt 31.34%. Ngược lại, kích cỡ S có tỷ lệ trả hàng cao nhất ở mức 5.65%.
*   **Lỗ hổng vận hành:** Với danh mục chủ lực là Streetwear, lý do khách hàng trả lại sản phẩm nhiều nhất là sai kích cỡ (`wrong_size`) với 7626 lượt.
*   **Tài chính & Thanh toán:** Vùng East là khu vực mang lại doanh thu cao nhất (hơn 7.63 tỷ VNĐ). Đối với các đơn bị hủy, phương thức thanh toán phổ biến nhất là Thẻ tín dụng (`credit_card`). Gói trả góp 6 tháng mang lại giá trị thanh toán trung bình cao nhất.
*   **Marketing:** Kênh có tỷ lệ thoát (bounce rate) thấp nhất là `email_campaign` (0.004458%). Có đến 38.66% tổng số dòng sản phẩm bán ra được áp dụng khuyến mãi.

---

## 3. Chẩn đoán Tổ chức & Chiến lược Đề xuất

Dự án áp dụng framework phân tích để bóc tách sự suy thoái của doanh nghiệp từ năm 2019:

### A. Nghịch lý "Doanh thu rỗng" (Hollow Revenue)
*   **Thực trạng:** Biểu đồ dòng tiền cho thấy dù tổng doanh thu (Top-line) tạo ra những đỉnh lớn, nhưng lợi nhuận gộp (Bottom-line) lại nằm sát trục 0 do chi phí giá vốn và khuyến mãi phình to. Phân tích lạm phát (Base year 2022) cho thấy sức mua thực tế đã giảm tới 40% trong giai đoạn 2012-2019.
*   **Bác bỏ giả thuyết COVID-19:** Dữ liệu chứng minh điểm uốn suy thoái bắt đầu ngay từ năm 2019, trước khi đại dịch xảy ra, do doanh nghiệp lạm dụng chiết khấu (tỷ lệ đơn dùng promo tăng đột biến từ 32% lên 45%).

### B. Vận hành & Chuỗi cung ứng
*   **Vấn đề:** Doanh nghiệp mất doanh thu cơ hội vì hiện tượng hết hàng (Stockout). Số ngày hết hàng có tương quan thuận cực mạnh (r = 0.65) với doanh thu, chứng tỏ công ty thường xuyên "cháy hàng" ngay trong các đỉnh mùa vụ.
*   **Chiến lược:** Đề xuất mô hình Pre-order (Zero Holding Cost) kết hợp công nghệ Fit-tech để loại bỏ chi phí lưu kho và giảm tỷ lệ hoàn hàng do sai kích cỡ.

### C. Tài chính & Mức độ Trung thành
*   **Vấn đề:** Khách hàng không còn trung thành với thương hiệu mà chỉ trung thành với voucher. Tỷ lệ dùng khuyến mãi có tương quan âm (r = -0.19) với doanh thu thuần. Việc đẩy tỷ lệ khuyến mãi lên vùng 60-70% đã khiến lợi nhuận gộp rơi tự do xuống mức âm.
*   **Chiến lược:** Đề xuất chiến lược "Gọng kìm kép" nhằm thanh lọc các tệp khách hàng "ký sinh" (phụ thuộc >50% vào khuyến mãi) và tái cấu trúc giá để đẩy biên lợi nhuận từ 13.8% lên 25.77%.

### D. Digital Marketing
*   **Vấn đề:** Có sự xuất hiện của "traffic rác" từ kênh `organic_search` khiến tỷ lệ chuyển đổi sụt giảm.
*   **Chiến lược:** Đề xuất cắt giảm 70% lưu lượng ảo từ kênh kém hiệu quả để tinh giản phễu tiếp thị, dự kiến giúp tối ưu hóa chi phí CAC từ 2,540 VNĐ xuống 2,138 VNĐ/đơn.

---

## 4. Kiến trúc Mô hình Dự báo 

Mô hình dự báo doanh thu và giá vốn cho giai đoạn 2023-2024 được thiết kế theo phương pháp phân rã đa tầng (Năm - Tháng - Ngày) nhằm kiểm soát chặt chẽ rủi ro tăng trưởng ảo:

*   **Cấp độ Năm (Macro-level):** Sử dụng thuật toán `ARIMA(1,1,0)` kết hợp với kỹ thuật `clipping` để giới hạn mức tăng trưởng an toàn trong biên độ ±5%. Giá vốn (COGS) được ràng buộc chặt chẽ với doanh thu thông qua tỷ suất lợi nhuận gộp trung bình của 3 năm gần nhất nhằm đảm bảo tính khả thi tài chính.
*   **Cấp độ Tháng (Seasonality):** Ứng dụng hồi quy Harmonic (Sine/Cosine terms) kết hợp với đường trung bình lịch sử (tỷ trọng 70/30) để tạo ra các trọng số mùa vụ mượt mà. Đặc biệt, thuật toán tích hợp khai thác chu kỳ đa dạng dựa trên các năm Chẵn/Lẻ (Odd/Even Year Pattern).
*   **Cấp độ Ngày (Micro-level):** Dữ liệu được nội suy về cấp ngày bằng cách tính toán hệ số (ratio) của từng ngày cụ thể so với mức trung bình của tháng đó trong quá khứ. Mô hình sau đó áp dụng thêm bộ lọc hiệu chỉnh theo Ngày-trong-tuần (Day-of-week correction), ví dụ: Thứ Hai (1.0453), Thứ Sáu (0.9426).
*   **Kết quả:** Quá trình dự báo đã xuất ra tệp `submission_NEUSolution.csv` chứa mức doanh thu tối ưu (Năm 2023: 1.413 tỷ VNĐ) cho hệ thống.