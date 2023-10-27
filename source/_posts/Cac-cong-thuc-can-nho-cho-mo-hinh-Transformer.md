---
title: "Các công thức cần nhớ cho mô hình Transformer"
date: 2023-10-27 23:00:00
excerpt: Các công thức tính số lượng tham số, chi phí tính toán, yêu cầu về bộ nhớ cho mô hình Transformer. Thường được dùng khi huấn luyện các mô hình LLM.
index_img: /images/arseny-togulev-uPuh-VwJRM0-unsplash-small.jpg
banner_img: /images/arseny-togulev-uPuh-VwJRM0-unsplash-medium.jpg
tags:
- Transformer
- Large Language Model
- Machine Learning
- Artificial Intelligence
---

# Tính số lượng tham số

## Công thức "gần" chính xác

{% katex '{ "displayMode": true }' %}
P = VH + L(12H^2 + 13H)
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
P = 12LH^2
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

# Tính yêu cầu bộ nhớ cho huấn luyện

Công thức tính yêu cầu bộ nhớ không cố định mà phụ thuộc vào các cài đặt khi huấn luyện mô hình, nhưng về cơ bản thì gồm có 4 thành phần chính:

{% katex '{ "displayMode": true }' %}
memory_{training} = memory_{model} + memory_{optimizer} + memory_{gradient} + memory_{activation}
{% endkatex %}

Trong đó:
- {% katex %}memory_{training}{% endkatex %}: bộ nhớ để huấn luyện mô hình
- {% katex %}memory_{model}{% endkatex %}: bộ nhớ để lưu trữ tham số của mô hình
- {% katex %}memory_{optimizer}{% endkatex %}: bộ nhớ để lưu trữ trạng thái của optimizer
- {% katex %}memory_{gradient}{% endkatex %}: bộ nhớ để lưu trữ giá trị gradient
- {% katex %}memory_{activation}{% endkatex %}: bộ nhớ để lưu trữ giá trị của các hàm activation

## Model Memory

Khi huấn luyện, có 3 loại dữ liệu để lưu trữ tham số của mô hình là fp32, fp16 và mixed-precision. Mỗi dạng sẽ có mức sử dụng bộ nhớ khác nhau:
- fp32:
{% katex '{ "displayMode": true }' %}
memory_{model} = 4P
{% endkatex %}

- fp16 và mixed-precision:
{% katex '{ "displayMode": true }' %}
memory_{model} = 2P
{% endkatex %}

## Optimizer Memory

Mỗi optimizer có mức sử dụng bộ nhớ khác nhau:
- AdamW:
{% katex '{ "displayMode": true }' %}
memory_{optimizer} = 12P
{% endkatex %}

- SGD:
{% katex '{ "displayMode": true }' %}
memory_{optimizer} = 8P
{% endkatex %}

- Bitsandbytes:
{% katex '{ "displayMode": true }' %}
memory_{optimizer} = 6P
{% endkatex %}

## Gradient Memory

Tương tự model, gradient cũng có thể được lưu dưới dạng fp32, fp16 hoặc mixed-precision:
- fp32:
{% katex '{ "displayMode": true }' %}
memory_{gradient} = 4P
{% endkatex %}

- fp16 và mixed-precision:
{% katex '{ "displayMode": true }' %}
memory_{gradient} = 2P
{% endkatex %}

## Activation Memory

{% katex '{ "displayMode": true }' %}
memory_{activation} = sbHL(10 + \frac {24} t + 5\frac {a*s} {H*t})
{% endkatex %}

Trong đó:
- s: độ dài của chuỗi đầu vào, tính bằng số token
- b: batch size
- H: số chiều của hidden state
- L: số lượng layer
- a: số attention head
- t: mức độ của tensor parallelism (nếu không dùng tensor parallelism thì t = 1)

Batch size khi huấn luyện các LLM thường rất lớn nên bộ nhớ sử dụng cũng lớn theo. Tuy nhiên, ta có thể giảm bộ nhớ sử dụng cho activation bằng cách tính lại giá trị của các hàm activation thay vì lưu trữ chúng, với cái giá phải trả là làm tăng chi phí tính toán. Cách làm này được gọi là "activation recomputation/checkpointing".

Có 2 cách để tính lại giá trị của các hàm activation:
- **Tính lại toàn bộ activation**, cách này sẽ giảm rất nhiều bộ nhớ sử dụng.
{% katex '{ "displayMode": true }' %}
memory_{activation} = 2sbHL
{% endkatex %}
- **Tính lại một phần activation**, cách này sẽ giảm bộ nhớ sử dụng ít hơn so với cách trên nhưng vẫn giảm được một phần.
{% katex '{ "displayMode": true }' %}
memory_{activation} = sbHL(10 + \frac {24} t)
{% endkatex %}

# Tài liệu tham khảo

- Counting the Number of Parameters in Deep Learning. Yekun's Note. https://ychai.uk/notes/2019/10/11/NN/Counting-the-number-of-parameters-in-deep-learning/
- Transformer Math 101. (2023). EleutherAI. https://blog.eleuther.ai/transformer-math/
- Touvron, Hugo & Lavril, Thibaut & Izacard, Gautier & Martinet, Xavier & Lachaux, Marie-Anne & Lacroix, Timothée & Rozière, Baptiste & Goyal, Naman & Hambro, Eric & Azhar, Faisal & Rodriguez, Aurelien & Joulin, Armand & Grave, Edouard & Lample, Guillaume. (2023). LLaMA: Open and Efficient Foundation Language Models. 10.48550/arXiv.2302.13971. 