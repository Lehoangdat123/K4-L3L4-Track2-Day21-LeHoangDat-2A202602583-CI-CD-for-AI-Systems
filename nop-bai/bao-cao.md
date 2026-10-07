# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Lê Hoàng Đạt |
| MSSV | 2A202602583 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/Lehoangdat123/K4-L3L4-Track2-Day21-LeHoangDat-2A202602583-CI-CD-for-AI-Systems |
| Ngày nộp | 2026/10/07 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Lần chạy 3 có F1 cao nhất (0.7149) và vượt ngưỡng 0.65, nên được chọn dù
accuracy cao nhất thuộc lần chạy 1 (0.8780). Điều này cho thấy accuracy không thay thế
được F1 khi cần đánh giá lớp thu nhập cao. Lần chạy 2 đạt F1 thấp hơn ngưỡng; vì nhiều
siêu tham số thay đổi cùng lúc, ba lần thử này chưa đủ để kết luận riêng tác động của
learning rate hay số cây.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Chỉ khoảng 24.8% mẫu thuộc lớp thu nhập trên 50K, nên mô hình luôn dự đoán “thu nhập
thấp” vẫn đạt accuracy xấp xỉ 75.2% nhưng bỏ sót toàn bộ lớp thu nhập cao và có F1 lớp
dương bằng 0. F1 lớp dương kết hợp precision và recall, thể hiện khả năng nhận diện lớp
thiểu số rõ hơn accuracy. Không dùng `average="weighted"` hoặc `"macro"` vì các giá trị
gộp đó không phản ánh trực tiếp chất lượng của lớp dương mà quality gate cần kiểm tra.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow SQLite không khởi động | SQLAlchemy 2.1 không tương thích với MLflow 2.13 đang dùng | Giới hạn SQLAlchemy dưới 2.1 trong `requirements.txt`. |
| Test tạo output dùng chung | Report và model được lưu theo thư mục làm việc hiện tại | Cho từng test đổi sang `tmp_path` để cô lập artifact. |
| Không xác minh được cloud deployment | Workspace chưa có bucket, secrets hoặc VM để chạy pipeline | Đã thêm DVC pointers và workflow; cần cấu hình secrets/tài nguyên cloud trước khi chạy thật. |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Với cùng bộ siêu tham số và holdout, F1 tăng 0.0205 còn accuracy tăng
0.0080 khi ghép batch 2. Đây là kết quả của lần chạy cục bộ, không khẳng định thêm dữ
liệu luôn cải thiện mô hình; pipeline cloud, deploy và health check chưa được xác minh
do chưa cấu hình tài nguyên/secrets.
