---
title: Triển khai GPT-J 6B Vietnamese News trên Docker và K8S
date: 2021-01-05 10:08:36
photos:
- /images/possessed-photography-U3sOwViXhkY-unsplash.jpg
tags:
- GPT-J
- Docker
- K8S
- Natural Language Generation
- Deep Learning
- Machine Learning
- Language Model
- Artificial Intelligence
---

Thời gian gần đây đang nổi lên 1 con AI tên là ChatGPT có khả năng giao tiếp như người thật, biết làm thơ, viết luận văn, giải toán, dịch tiếng Anh sang tiếng Việt, ngay cả viết và đọc hiểu code. Thậm chí nó có thể "giả lập" máy ảo linux và "chat với 1 ChatGPT khác trong máy ảo đó" (Đọc thêm ở [đây](https://www.engraved.blog/building-a-virtual-machine-inside/)). Bản chất bên dưới của hệ thống này là 1 mô hình Generative Pre-trained Transformer 3 (GPT-3) với số lượng tham số khổng lồ (175 tỷ), chỉ riêng việc lưu mô hình này thôi cũng cần đến 800 GB! Ngay cả khi OpenAI công bố mô hình này cho cộng đồng thì cũng không thể tự sử dụng được với kinh phí khiêm tốn.

May mắn là có một mô hình GPT khác đến từ cộng đồng AI Việt Nam, tên là [GPT-J 6B Vietnamese News](https://huggingface.co/VietAI/gpt-j-6B-vietnamese-news), được train bởi VietAI. Mô hình này chỉ có 6 tỷ tham số, đủ nhỏ để chạy trên những con GPU giá rẻ. Trong bài viết này, mình sẽ hướng dẫn cách triển khai mô hình này lên môi trường Docker và K8S, sử dụng thông qua API để lấy kết quả.

Link tới Github repo chứa code trong bài viết này: [https://github.com/duydvu/gpt-j-6B-vietnamese-news-api](https://github.com/duydvu/gpt-j-6B-vietnamese-news-api)

<escape><!-- more --></escape>

# Load và chạy thử mô hình

Bước đầu tiên là load tokenizer và tham số của mô hình:

```python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM

# Load tokenizer
tokenizer = AutoTokenizer.from_pretrained(model_path)

# Load tham số mô hình
model = AutoModelForCausalLM.from_pretrained(model_path,
                                             torch_dtype=torch.float16,
                                             low_cpu_mem_usage=True)
model.parallelize() # Chia nhỏ mô hình và phân bổ lên nhiều GPU để tránh bị lỗi OOM
```

Trong đoạn code trên, mình thực hiện 2 phương pháp tối ưu:
1. Sử dụng kiểu dữ liệu float16 thay vì float32 để giảm dung lượng bộ nhớ trên GPU. Vì float16 tương ứng với 2 byte còn float32 tới 4 byte nên tiết kiệm được 50% dung lượng.
2. Vì mô hình này vẫn rất lớn so với dung lượng bộ nhớ của các loại GPU phổ biến, nếu bạn có nhiều GPU thì có thể phân bổ các tham số của mô hình trên nhiều GPU thay vì chỉ chạy trên 1 con. Phương pháp này có tên là **Model Parallel**.

Sau khi đã xong bước 1, chạy thử mô hình để kiểm tra xem có bị lỗi gì không:

```python
# Convert đoạn text từ dạng string sang dãy số integer 
input_ids = tokenizer.encode("Tiềm năng của trí tuệ nhân tạo", return_tensors='pt').to('cuda:0')

# Chạy mô hình
outputs = model.generate(
  input_ids,
  max_length=256,
  do_sample=True,
  top_k=50,
  top_p=0.9,
  num_return_sequences=1,
)

# Decode kết quả của mô hình từ dãy số integer sang dạng string và in ra màn hình
for output in outputs:
  print(tokenizer.decode(output, skip_special_tokens=True))
```

Vì thuật toán sinh văn bản mà mình đang sử dụng là lấy mẫu dựa trên xác suất nên sau mỗi lần chạy, mô hình sẽ cho ra kết quả khác nhau. Kết quả thu được trên máy của mình là:

```
Tiềm năng của trí tuệ nhân tạo là rất lớn, nhiều ứng dụng có thể ra đời trong tương lai không xa, và điều đó đòi hỏi sự nỗ lực của cả cộng đồng.

Một điểm khác cần lưu ý là trí tuệ nhân tạo là một lĩnh vực phát triển hoàn toàn mới và không thể chỉ dựa vào mỗi việc nghiên cứu của các trường, viện mà phải kết hợp với các doanh nghiệp, và phải có sự kết nối mạnh mẽ với các doanh nghiệp.

Về việc chuẩn bị nhân lực cho cuộc cách mạng công nghiệp lần thứ 4, Phó Thủ tướng lưu ý các trường cần có sự đổi mới về cơ chế, phương pháp, mô hình dạy và học nhằm tạo môi trường tốt nhất cho trí tuệ nhân tạo.

"Nếu cứ làm theo cách cũ thì các trường sẽ như "rúc thóc" vào bao, nhưng nếu áp dụng phương pháp mới mà các trường chưa có, mới ở giai đoạn đầu mà có quy mô lớn hơn nhiều lần thì sẽ rất khó khăn", Phó Thủ tướng chia sẻ.

Phó Thủ tướng Vũ Đức Đam đề nghị Bộ KH&CN tiếp tục có sự phối hợp với các bộ, ngành, trước hết là Bộ KH&CN xây dựng chiến lược, quy hoạch phát triển, thực hiện đề án tăng cường.
```

Sau khi chạy thử thành công, mình viết lại đoạn code trên thành 1 class trong file `src/predictor.py` và đóng gói toàn bộ code bao gồm cả phần API server vào trong thư mục `src`.
