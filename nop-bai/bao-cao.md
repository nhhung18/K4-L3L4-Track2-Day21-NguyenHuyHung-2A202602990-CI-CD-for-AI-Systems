# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Huy Hùng |
| MSSV | 2A202602990 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/nhhung18/K4-L3L4-Track2-Day21-NguyenHuyHung-2A202602990-CI-CD-for-AI-Systems.git |
| Ngày nộp | 7/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.878 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.846 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.874 |
| 4 | 150 | 0.2 | 4 | 0.6912 | 0.866 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Bộ tham số này đạt f1_score cao nhất (0.7149) trên tập holdout, vượt ngưỡng chất lượng 0.65 với biên an toàn lớn. Đáng chú ý, lần chạy có accuracy cao nhất (lần 1, accuracy=0.878) lại không phải lần có f1_score cao nhất — điều này cho thấy accuracy không phản ánh đúng khả năng phát hiện lớp thu nhập cao. Về đánh đổi giữa n_estimators và learning_rate: khi giảm learning_rate xuống 0.05 (lần 2), dù giữ max_depth nhỏ, f1_score tụt xuống chỉ còn 0.6051 vì mỗi cây đóng góp quá ít và số lượng cây (50) không đủ bù lại; tăng n_estimators lên 200 kết hợp learning_rate=0.1 cho kết quả tốt nhất.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult có phân bố lớp mất cân bằng: chỉ 24.8% mẫu thuộc lớp thu nhập cao (target=1). Hệ quả là một mô hình luôn trả lời "thu nhập thấp" cho mọi mẫu sẽ đạt accuracy khoảng 0.752 — con số trông khá cao nhưng mô hình đó hoàn toàn vô dụng vì không phát hiện được bất kỳ trường hợp thu nhập cao nào (f1_score=0.000). Accuracy đo tỷ lệ dự đoán đúng trên toàn bộ mẫu, trong đó lớp đa số (thu nhập thấp) chiếm ưu thế và kéo chỉ số lên cao một cách giả tạo. F1-score của lớp dương đo sự cân bằng giữa precision và recall riêng cho lớp thu nhập cao, buộc mô hình phải thực sự học được đặc trưng của lớp thiểu số. Không dùng `average="weighted"` hay `average="macro"` vì các giá trị đó bị lớp đa số kéo lên và không phản ánh đúng chất lượng phát hiện lớp thu nhập cao — mục tiêu thực sự của bài toán.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow 2.13.0 không cài được trên Python 3.13 | Requirements.txt pin version cũ không tương thích | Đổi sang `mlflow>=3.0.0` bỏ pin cứng version |
| `UntrustedTypesFoundException` khi log model sklearn | MLflow 3.x dùng skops thay pickle, yêu cầu khai báo trusted types | Thêm `skops_trusted_types=["sklearn.tree._tree.Tree"]` vào `log_model` |
| SSH deploy key không được nhận dạng trong GitHub Actions | Newline bị mất khi copy-paste private key vào GitHub Secrets | Encode key thành base64 một dòng, lưu vào secret và decode trong workflow |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.874 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.882 |

**Nhận xét:** F1-score tăng từ 0.7149 lên 0.7354 khi gấp đôi dữ liệu huấn luyện (22.361 → 44.722 mẫu), cho thấy dữ liệu bổ sung giúp mô hình học tốt hơn dù hai batch có cùng phân phối. Tuy nhiên mức tăng tương đối nhỏ (0.02), phù hợp với lý thuyết: khi dữ liệu mới không mang thêm phân phối mới, lợi ích giảm dần theo quy luật diminishing returns. Điều quan trọng được kiểm chứng ở Bước 3 là quy trình tự động chạy đúng — chỉ một lần `git push` dữ liệu đã đi hết vòng từ commit đến model mới được phục vụ trên VM mà không cần can thiệp thủ công.

