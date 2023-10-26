---
title: "Các công thức cần nhớ cho mô hình Transformer"
date: 2023-10-24 22:00:00
excerpt: Các công thức cần nhớ cho mô hình Transformer
index_img: /images/brandon-morgan-3qucB7U2l7I-unsplash-small.jpg
banner_img: /images/brandon-morgan-3qucB7U2l7I-unsplash-medium.jpg
tags:
- Transformer
- Large Language Model
- Machine Learning
- Artificial Intelligence
---

# Tính số lượng tham số

## Công thức "gần" chính xác

{% katex '{ "displayMode": true }' %}
P = V * H + L * (12 * H^2 + 13 * H)
{% endkatex %}

Trong đó:
- P: số lượng tham số của mô hình
- V: số lượng từ trong từ điển
- H: số chiều của hidden state
- L: số lượng layer

Công thức này "gần" chính xác vì nó không tính đến số lượng tham số của các layer normalization và các layer khác. Hơn nữa, các LLM hiện nay có 1 số thay đổi nhỏ trên kiến trúc gốc của Transformer (LLaMA, GPT) nên kết quả của công thức này cũng không chính xác 100%.

Ví dụ với mô hình [PhoBERT-base](https://github.com/VinAIResearch/PhoBERT)
```python
from transformers import AutoModel
model = AutoModel.from_pretrained('vinai/phobert-base')
sum(p.numel() for p in model.parameters())
```

Chạy đoạn code trên sẽ cho ta con số chính xác là 134 998 272.

Áp dụng công thức trên với V = 64001, H = 768, L = 12:

{% katex '{ "displayMode": true }' %}
P = 64001 * 768 + 12 * (12 * 768^2 + 13 * 768) = 134207232
{% endkatex %}

Như vậy thì con số chính xác cao hơn kết quả của công thức 791 040 tham số, tương đương 0.58% tỷ lệ sai số.

## Công thức xấp xỉ

Đối với các LLM với hàng tỷ tham số như hiện nay thì {% katex %}H^2{% endkatex %} lớn hơn nhiều so với {% katex %}H{% endkatex %} nên ta có thể đơn giản hoá công thức trên để tính xấp xỉ như sau:

{% katex '{ "displayMode": true }' %}
P = 12 * L * H^2
{% endkatex %}

Nếu áp dụng công thức này với PhoBERT-base thì ta sẽ nhận được kết quả là 84M tham số, tương đương 37% tỷ lệ sai số. Sai số lớn này là do đối với các mô hình nhỏ thì tỷ lệ tham số của ma trận token embeddings còn khá lớn.

Thử áp dụng với mô hình LLaMA, ta có được bảng sau:


| Mô hình | H | L | P xấp xỉ | P chính xác | Tỷ lệ sai số |
|---|:---:|:---:|:---:|:---:|:---:|
| LLaMA 7B | 4096 | 32 | 6.4B | 6.7B | 4.4% |
| LLaMA 13B | 5120 | 40 | 12.6B | 13.0B | 3.2% |
| LLaMA 33B | 6656 | 60 | 31.9B | 32.5B | 1.8% |
| LLaMA 65B | 8192 | 80 | 64.4B | 65.2B | 1.2% |

Như vậy thì khi mô hình càng lớn thì tỷ lệ sai số càng giảm.

# Tính chi phí cho huấn luyện

Chi phí tính toán để huấn luyện mô hình Transformer, không bao gồm các phương pháp PEFT như LoRA.

{% katex '{ "displayMode": true }' %}
C \approx \tau T = 6PD
{% endkatex %}

Trong đó:
- C: chi phí tính toán, tính bằng Floating Point Operations
- {% katex %}\tau{% endkatex %}: tổng sức mạnh tính toán của hệ thống, tính bằng FLOPs
- T: Thời gian huấn luyện, tính bằng giây
- P: số lượng tham số của mô hình
- D: kích thước tập dữ liệu, tính bằng số token

# Tính yêu cầu về bộ nhớ

