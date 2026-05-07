# Báo Cáo Lab CI/CD cho AI Systems

**Họ tên:** Vũ Tiến Thành
**Repo:** https://github.com/TienThanh-dev/Day21-Track2-CI-CD-for-AI-Systems
**Ngày:** 2026-05-07

---

## 1. Tổng Quan Kiến Trúc

Xây dựng hệ thống MLOps hoàn chỉnh từ thực nghiệm cục bộ đến triển khai tự động trên AWS:

```
GitHub Actions (CI/CD)
    ├── Unit Test (pytest)
    ├── Train (DVC pull data → train → S3 upload model)
    ├── Eval Gate (accuracy >= 0.70)
    └── Deploy (SSH restart service)
            │
            v
[AWS S3: lab21-cicd]          [AWS EC2: mlops-serve]
  data/dvc/                         FastAPI (port 8000)
  models/latest/model.pkl            POST /health
                                    POST /predict
```

---

## 2. Kết Quả Từng Bước

### Bước 1 — MLflow Tracking

Chạy 3 thí nghiệm với các siêu tham số khác nhau trên máy cục bộ, ghi nhận kết quả trong MLflow.

### Bước 2 — CI/CD Pipeline

| Job | Trạng thái |
|---|---|
| Unit Test | ✓ Pass |
| Train | ✓ Pass (accuracy = 0.762) |
| Eval Gate | ✓ Pass (0.762 >= 0.70) |
| Deploy | ✓ Pass |

### Bước 3 — Huấn luyện liên tục

Pipeline tự động kích hoạt khi commit file `.dvc` thay đổi.

---

## 3. Bộ Siêu Tham Số Tối Ưu

Chọn bộ siêu tham số tốt nhất dựa trên kết quả MLflow:

| Tham số | Giá trị |
|---|---|
| n_estimators | 500 |
| max_depth | 50 |
| min_samples_split | 5 |

**Lý do chọn:**
- Tăng `n_estimators` từ 100 → 500 cải thiện accuracy nhờ ensemble lớn hơn.
- Tăng `max_depth` từ 5 → 50 cho phép cây học sâu hơn, phù hợp với dữ liệu wine quality có 12 đặc trưng.
- `min_samples_split=5` giảm overfitting nhẹ so với giá trị mặc định 2.

---

## 4. Kết Quả Huấn Luyện

| Chỉ số | Giá trị |
|---|---|
| Accuracy | 0.762 |
| F1 Score | 0.761 |

Accuracy vượt ngưỡng eval gate (0.70), pipeline tự động triển khai.

---

## 5. Khó Khăn và Cách Giải Quyết

| Vấn đề | Cách giải quyết |
|---|---|
| MLflow `log_model` lỗi với sqlite URI | Dùng `mlflow.set_tracking_uri` thay vì env var; set artifact root thành local folder |
| DVC S3 cần credentials | Cài `dvc[s3]` và chạy `aws configure` để tạo credentials file |
| SSH key GitHub Actions không nhận diện | Đảm bảo secret VM_SSH_KEY copy toàn bộ private key (không cắt, không thừa dòng trắng) |
| EC2 không có fastapi/scikit-learn | Tạo virtual environment (`python3 -m venv ~/venv`) và cài packages qua venv |
| Service EC2 không chạy vì credentials | Copy `~/.aws/credentials` lên EC2, set `AWS_SHARED_CREDENTIALS_FILE` trong systemd service |
| EC2 port 8000 chặn connection | Mở inbound rule Custom TCP port 8000 trong Security Group |

