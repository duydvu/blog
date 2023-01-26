---
title: Triển khai Triton Inference Server trên Google Kubernetes Engine
date: 2023-01-26 20:42:00
excerpt: Trong bài viết này, mình sẽ hướng dẫn bạn cách triển khai Triton trên GKE để chạy các mô hình Deep Learning có sử dụng GPU.
index_img: /images/scott-webb-3LsocYqXWpM-unsplash-small.jpg
banner_img: /images/scott-webb-3LsocYqXWpM-unsplash-large.jpg
tags:
- Triton Inference Server
- K8S
- Google Kubernetes Engine
- MLOps
- Model Serving
- Deep Learning
- Machine Learning
- Artificial Intelligence
---

Khi triển khai mô hình Deep Learning lên môi trường K8S, chúng ta có thể áp dụng phương pháp giống như đối với các chương trình thông thường, về cơ bản thì sẽ gồm những bước sau:
1. Đóng gói code và mô hình vào Docker image.
2. Đẩy image lên registry.
3. Tạo deployment và service trên K8S.
4. Tạo ingress nối đến service vừa tạo.

Phương pháp này rất đơn giản và hiệu quả đối với những mô hình gọn nhẹ không cần đến GPU, ví dụ như mô hình phân loại văn bản bằng LSTM, các mô hình Machine Learning truyền thống như SVM hay Random Forest. Nhưng khi mô hình lớn đến mức không đáp ứng được các yêu cầu về latency hoặc throughput thì triển khai lên GPU là điều cần thiết. Nếu chúng ta vẫn áp dụng phương pháp kể trên thì có những nhược điểm như sau:
1. Các image dùng được GPU thường có kích thước lớn, cộng với việc mô hình cũng có kích thước lớn khiến cho **image đầu ra nặng đến mức làm chậm đáng kể thời gian push và pull image trong quá trình triển khai**. Điều này gây khó khăn nếu mô hình thường xuyên được cập nhật với dữ liệu mới.
2. **Kubernetes không cung cấp cơ chế native cho việc cấp phát 1 GPU cho nhiều container**. Không giống như khi cấp phát CPU, container khi yêu cầu K8S cấp phát GPU chỉ có thể được cấp nguyên cả 1 GPU chứ không được yêu cầu 0.5 hay 0.1 GPU.
