---
title: "Quick Note #1: LoRA cho bài toán fine-tuning LLM"
date: 2023-05-28 21:06:00
excerpt: Ứng dụng Low-Rank Adaptation (LoRA) để fine-tune mô hình ngôn ngữ lớn (LLM) với chi phí thấp.
index_img: /images/vlado-paunovic-iBG594vhR1k-unsplash-small.jpg
banner_img: /images/vlado-paunovic-iBG594vhR1k-unsplash-large.jpg
tags:
- LoRA
- LLM
- Deep Learning
- Machine Learning
- Artificial Intelligence
---

# Low-Rank Adaptation
- Phần lớn tính toán trong các mô hình ngôn ngữ lớn nói riêng và các mô hình Deep Learning nói chung là những phép nhân ma trận. Nếu tối ưu được quá trình này thì sẽ giảm được chi phí đáng kể.
- Bản chất của quá trình fine-tune là tạo ra 1 bộ tham số {% katex %}\Delta W{% endkatex %} để cộng nó vào {% katex %}W_0{% endkatex %} - bộ tham số đã được pre-trained - để tạo ra bộ tham số mới {% katex %}W = W_0 + \Delta W{% endkatex %}.
- Giả thuyết được đặt ra: **{% katex %}\Delta W{% endkatex %} là một ma trận có rank (hạng) thấp**.
- Rank của ma trận càng nhỏ thì số lượng tham số cần tối ưu càng ít. Suy ra, cần ít bộ nhớ để lưu trữ và tài nguyên để tính toán hơn.
- Công thức trên có thể được biểu diễn lại bằng {% katex %}W = W_0 + \Delta W = W_0 + BA{% endkatex %} với {% katex %}B{% endkatex %} và {% katex %}A{% endkatex %} là 2 ma trận có số chiều thấp hơn nhiều so với {% katex %}W_0{% endkatex %}.
- Ví dụ: {% katex %}W_0 \in \mathbb{R}^{1000 \times 1000}{% endkatex %}, {% katex %}B \in \mathbb{R}^{1000 \times 10}{% endkatex %}, {% katex %}A \in \mathbb{R}^{10 \times 1000}{% endkatex %} thì số lượng tham số cần tối ưu là {% katex %}1000 \times 10 + 10 \times 1000 = 20000{% endkatex %} thay vì {% katex %}1000 \times 1000 = 1000000{% endkatex %} như trước đây. Tức là giảm đi 98% số lượng tham số cần tối ưu!
- Khi triển khai mô hình LoRA, cần phải tải {% katex %}W_0{% endkatex %} lẫn {% katex %}B{% endkatex %} và {% katex %}A{% endkatex %} vào bộ nhớ và thực hiện phép toán {% katex %}W = W_0 + BA{% endkatex %} để sinh ra bộ tham số cho mô hình đã được fine-tune.
- Nhược điểm:
  - Mỗi mô hình LoRA chỉ có thể xử lý được 1 tác vụ tại 1 thời điểm. Khi cần thực hiện tác vụ khác thì phải thực hiện chuyển đổi bộ tham số.
  - Giả sử có nhiều mô hình LoRA được triển khai, nhưng chỉ có 1 instance của {% katex %}W_0{% endkatex %} được tải vào bộ nhớ. Nếu dữ liệu đầu vào buộc mô hình phải chuyển đổi qua lại giữa các LoRA khác nhau thì sẽ làm tăng chi phí tính toán và giảm hiệu năng của mô hình.
- Giải pháp: Dùng [Prompt Tuning](https://arxiv.org/abs/2104.08691).

# Tài liệu tham khảo
- [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/pdf/2106.09685.pdf)
