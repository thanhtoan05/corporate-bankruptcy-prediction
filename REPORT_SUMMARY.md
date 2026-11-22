# Tóm tắt phân tích rủi ro phá sản doanh nghiệp

**Mục đích.** Xếp hạng rủi ro từ báo cáo tài chính và minh họa cách lập danh sách rà soát theo chi phí. Kết quả là đánh giá hồi cứu, chưa phải bằng chứng về hiệu quả triển khai tín dụng.

**Dữ liệu và phương pháp.** Bộ Kaggle gồm 78.682 doanh nghiệp–năm của 8.971 doanh nghiệp Mỹ, giai đoạn 1999–2018. Nhãn t+1 được suy ra từ năm quan sát cuối của công ty failed; dataset không cung cấp ngày phá sản để xác minh. Train 1999–2011, valid 2012–2014, test 2015–2018. Test có 119 sự kiện trên 12.282 dòng, tỷ lệ nền 0,97%. Chọn mô hình bằng Average Precision trên valid; Platt học từ dự báo ngoài mẫu theo thời gian trong train.

**Kết quả test.**

| Mô hình | ROC-AUC | PR-AUC (Average Precision) |
|---|---:|---:|
| Altman Z cố định | 0,795 | 0,026 |
| Logistic trên 5 biến Altman | 0,865 | 0,040 |
| Logistic đầy đủ | 0,906 | 0,192 |
| Random Forest | 0,926 | 0,176 |
| XGBoost | 0,915 | 0,224 |

XGBoost được chọn theo valid và có PR-AUC test khoảng 23 lần mức tham chiếu 0,0097. Tuy nhiên, khoảng tin cậy ghép cặp 95% của chênh PR-AUC XGBoost − Logistic là **[−0,044; 0,114]**: chưa đủ bằng chứng về ưu thế giữa hai mô hình. Refit XGBoost đến 2014 tăng PR-AUC khoảng 0,019, với khoảng ghép cặp **[−0,013; 0,048]**, nên cũng chưa kết luận cải thiện theo chỉ số này. Bootstrap giữ mô hình cố định, chưa bao gồm toàn bộ bất định huấn luyện.

**Xác suất và hành động.** Hiệu chuẩn ngoài mẫu trên train dự báo tổng khoảng **183 sự kiện**, so với **119 thực tế**; xác suất bình quân cao hơn tỷ lệ quan sát ở cả bốn năm test. Vì vậy chưa thể coi xác suất là PD đã hiệu chuẩn tốt cho vận hành. Kịch bản rà soát giả định chi phí 1 triệu VND/hồ sơ, EAD 1 tỷ, LGD 45%, hiệu quả tránh tổn thất 10% cho ngưỡng valid **4,60%**. Trên test: TP 83, FP 925, FN 36, TN 11.238; Recall 69,7%, Precision 8,2%. Lợi ích ròng ước tính 2,727 tỷ VND chỉ là kết quả kịch bản, không phải lợi nhuận hay hiệu quả can thiệp đã được đo.

**Kiểm tra bổ sung.** Backtest 2012–2018 cho PR-AUC XGBoost dao động 0,104–0,343; chất lượng thay đổi theo năm. Logistic gọn có chín biến và VIF tối đa 3,22 nhưng PR-AUC test giảm còn 0,033. XGBoost đơn điệu đạt 0,243, song khoảng chênh lệch với bản gốc chứa 0. Notebook còn có stress test, nhãn hai năm, cờ thiếu và PSI/CSI. Các bảng ở `results/` là kiểm tra độ vững, không dùng để chọn lại mô hình chính theo test.

**Việt Nam và giới hạn.** Chỉ có 105 doanh nghiệp–năm của 16 mã thép, thiếu nhiều kỳ và chưa có sự kiện xác minh. Chuyển thành phụ lục: X năm t, nhãn từ t+1/t+2; train còn 31 dòng/1 ca, holdout 14 dòng/2 ca. Không chọn winner, tối ưu ngưỡng, kiểm định Hosmer–Lemeshow hoặc so sánh Mỹ–Việt. Vẫn thiếu ngày công bố BCTC, ngành và dữ liệu khoản vay thực. Test đã được xem trong nghiên cứu; cần tập ngoài thời gian mới và sự kiện xác minh trước triển khai.
