---
title: Triển khai GPT-J 6B Vietnamese News trên Docker và K8S
date: 2022-12-28 11:28:00
index_img: /images/possessed-photography-U3sOwViXhkY-unsplash.jpg
banner_img: /images/possessed-photography-U3sOwViXhkY-unsplash.jpg
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

Trước khi chạy thử mô hình này, bạn cần đảm bảo đủ bộ nhớ GPU để load mô hình lên máy. Yêu cầu tối thiểu là khoảng 17 GB. Nếu bạn có nhiều hơn 1 GPU và tổng bộ nhớ nhiều hơn con số này thì vẫn có thể chạy được. Mình đã thử thành công trên 2 card GTX 1080Ti 12GB.

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

```text
Tiềm năng của trí tuệ nhân tạo là rất lớn, nhiều ứng dụng có thể ra đời trong tương lai không xa, và điều đó đòi hỏi sự nỗ lực của cả cộng đồng.

Một điểm khác cần lưu ý là trí tuệ nhân tạo là một lĩnh vực phát triển hoàn toàn mới và không thể chỉ dựa vào mỗi việc nghiên cứu của các trường, viện mà phải kết hợp với các doanh nghiệp, và phải có sự kết nối mạnh mẽ với các doanh nghiệp.

Về việc chuẩn bị nhân lực cho cuộc cách mạng công nghiệp lần thứ 4, Phó Thủ tướng lưu ý các trường cần có sự đổi mới về cơ chế, phương pháp, mô hình dạy và học nhằm tạo môi trường tốt nhất cho trí tuệ nhân tạo.

"Nếu cứ làm theo cách cũ thì các trường sẽ như "rúc thóc" vào bao, nhưng nếu áp dụng phương pháp mới mà các trường chưa có, mới ở giai đoạn đầu mà có quy mô lớn hơn nhiều lần thì sẽ rất khó khăn", Phó Thủ tướng chia sẻ.

Phó Thủ tướng Vũ Đức Đam đề nghị Bộ KH&CN tiếp tục có sự phối hợp với các bộ, ngành, trước hết là Bộ KH&CN xây dựng chiến lược, quy hoạch phát triển, thực hiện đề án tăng cường.
```

Sau khi chạy thử thành công, mình viết lại đoạn code trên thành 1 class trong file `src/predictor.py` và đóng gói toàn bộ code bao gồm cả phần API server vào trong thư mục `src`.

# Tạo Docker image

Để build được Docker, trước hết bạn cần phải tải mô hình về máy theo các bước sau:
1. Cài đặt [git-lfs](https://git-lfs.com/)
2. Chạy lệnh `git clone https://huggingface.co/VietAI/gpt-j-6B-vietnamese-news`

Lệnh clone sẽ tải mô hình về và đặt ở thư mục hiện tại trên terminal.

Sau khi tải xong, dùng lệnh `mv` để đưa thư mục vừa clone về vào thư mục của repo API:
```bash
mv ./gpt-j-6B-vietnamese-news ./gpt-j-6B-vietnamese-news-api
```

Cuối cùng là dùng lệnh `build` Docker image:
```bash
docker build -t gpt-j .
```

Tiến hành chạy thử bằng việc tạo 1 container và map tới port 5000 của máy:
```bash
docker run -it --rm -p 5000:5000 gpt-j
```

Gọi thử bằng lệnh `curl`:
```bash
curl --location --request POST 'http://localhost:5000/predict' \
  --header 'Content-Type: application/json' \
  --data-raw '{
    "text": "Tiềm năng của trí tuệ nhân tạo",
    "n_samples": 1
  }'
```

# Triển khai lên môi trường K8S với Helm Chart

## Một chút phân tích

Trong bài viết này, mình chọn Google Cloud Platform (GCP) là nhà cung cấp dịch vụ cloud để triển khai mô hình lên K8S vì mình thường xuyên dùng GCP trong công việc. Dù vậy, nếu bạn có sử dụng nhà cung cấp khác thì cũng không thành vấn đề, quan trọng là họ có GPU cho K8S là được.

Sau khi nghiên cứu các mức giá GPU của GCP, mình quyết định chọn Nvidia T4 bởi 2 tiêu chí:
1. Mức giá - giá thuê hàng tháng của T4 là gần 180 USD, nếu dùng spot VM thì giá chỉ còn 80 USD, cộng với chi phí của CPU, RAM và disk thì khoảng 90 USD, thấp thứ 2 chỉ sau dòng K80.
2. Tốc độ xử lý - Lý do mình không chọn K80 là vì nó không hỗ trợ FP16 ([nguồn tham khảo](https://docs.nvidia.com/deeplearning/tensorrt/support-matrix/index.html#hardware-precision-matrix)), giúp giảm bộ nhớ cũng như tăng tốc độ chạy mô hình.

Với dung lượng 16 GB, cần tới 2 card T4 thì mới có thể chạy được mô hình GPT-J này. Như vậy chi phí hàng tháng nếu duy trì liên tục là 90 x 2=180 USD. Nếu chỉ thỉnh thoảng chạy thì chi phí sẽ còn thấp hơn nữa.

## Bắt đầu triển khai

Helm là một công cụ rất tuyệt vời cho việc đóng gói ứng dụng mà chúng ta vừa tạo để đưa lên K8S. Mình đã tạo Helm Chart và để trong thư mục `k8s/chart`. Chart này bao gồm 2 thành phần:

1. Deployment: Tạo deployment chứa pod chạy container gpt-j.
2. Service: Tạo service kết nối tới pod trong deployment ở trên.

Trong phần deployment có 2 phần quan trọng cần lưu ý, một là:
```YAML
strategy:
  rollingUpdate:
    maxSurge: 0
    maxUnavailable: 1
  type: RollingUpdate
```

Mặc định trong K8S khi cập nhật deployment thì sẽ tạo thêm 1 pod mới chạy song song với pod cũ, nhưng trong trường hợp này mình chỉ có đủ GPU để chạy cho 1 pod nên K8S sẽ không thể thay thế được pod cũ vì không có GPU để chạy pod mới. Vì vậy mình thay đổi strategy để khi cập nhật deployment thì K8S sẽ xóa pod cũ trước khi tạo pod mới. Như vậy thì sẽ không bị hiện tượng trên, nhưng sẽ bị vấn đề khác là xảy ra downtime trong quá trình deploy.

Hai là:

```YAML
resources:
  requests:
    cpu: 500m
    memory: 8000Mi
    nvidia.com/gpu: "2"
  limits:
    memory: 16000Mi
    nvidia.com/gpu: "2"
```

Có 2 dòng quan trọng là `nvidia.com/gpu: "2"` dùng để cấp phát tài nguyên GPU cho pod. Bắt buộc phải có 2 dòng này thì pod mới có thể dùng GPU được. Một điểm trừ là hiện tại GCP chỉ cho phép 1 GPU được cấp phát tới 1 pod nên nhiều pod không thể chia sẽ cùng 1 GPU được. Nếu có nhiều mô hình khác cần dùng tới GPU thì tốt nhất bạn nên dùng các giải pháp chạy mô hình tập trung như [Triton](https://github.com/triton-inference-server/server) để có thể chạy nhiều mô hình trên cùng 1 GPU.

Bước cuối cùng là cài đặt Helm Chart này lên K8S thông qua lệnh sau:

```bash
helm install --set namespace=default --set image=gpt-j --set version=latest gpt-j ./k8s/chart
```

Như vậy là chúng ta đã thành công trong việc triển khai GPT-J lên K8S rồi đó. Chúc bạn cũng thành công như mình nhé!
