---
title: 'Làm quen với Kafka: Phần 1 - Chat với nhau trên console'
date: 2021-01-01 18:18:04
photos:
- /images/nicolas-j-leclercq-qDLLP0yP7FU-unsplash.jpg
tags:
- Kafka
- Coding
- Python
---

Trong 1 hệ thống theo kiến trúc Microservice, để xử lý 1 khối lượng dữ liệu lớn với sự tác động của nhiều service khác nhau, ta phải có cơ chế để các service này giao tiếp với nhau một cách hiệu quả, Kafka được sinh ra để làm nhiệm vụ đó. Trong phần đầu tiên của series "Làm quen với Kafka", mình sẽ viết 1 ứng dụng Python dùng để chat với nhau trên màn hình console để hiểu được những tính năng cơ bản của Kafka.

Lưu ý đây chỉ là 1 ứng dụng làm cho vui để làm quen với Kafka thôi chứ không nên đưa vào thực tế nhé.

# Trước hết, Kafka là gì?

Kafka là 1 nền tảng event streaming rất phổ biến hiện nay được phát triển bởi tổ chức Apache và là một phần mềm mã nguồn mở. Event streaming ở đây có nghĩa là việc lấy dữ liệu theo thời gian thực từ những nguồn như là cơ sở dữ liệu, cảm biến, thiết bị di động, dịch vụ đám mây và ứng dụng dưới dạng những luồng sự kiện; lưu những sự kiện này để sau đó có thể lấy lên lại và xử lý.

## Nghe phức tạp quá. Tìm 1 ví dụ cho dễ hiểu nào!

Tưởng tượng bạn đang xem kênh Discovery trên TV của bạn. Những hình ảnh và âm thanh mà bạn thấy và nghe được đều được phát đi từ đài truyền hình của kênh này. Ở đây, đài truyền hình của kênh Discovery đóng vai trò là **publisher**, còn bạn - người xem là **subscriber** và kênh Discovery chính là **topic**.

Publisher có nhiệm vụ là phát đi thông tin vào 1 topic (kênh), thông tin trong trường hợp này là dữ liệu ở dạng hình ảnh và âm thanh. Subscriber có thể đăng ký để lắng nghe trên 1 topic (chọn kênh để xem) và nhận được thông tin mà publisher phát đi. Subscriber cũng có thể lắng nghe trên nhiều topic (xem nhiều kênh trên nhiều TV, nếu bạn giàu và rảnh :D).

Đối với publisher, việc ai nhận được thông tin không quan trọng, miễn là cứ có thông tin thì nó sẽ phát đi vào 1 hoặc nhiều topic cụ thể. Còn đối với subscriber, publisher nào phát không quan trọng, miễn là topic đó có thông tin thì cứ lấy ra mà xử lý.

# Viết ứng dụng chat trên console

Kafka hoạt động theo cơ chế client-server, trong đó, server là 1 process chạy trên 1 máy tính và client có thể kết nối đến nó giống như cách bạn kết nối đến database, sử dụng địa chỉ IP và port cùng với username và password nếu có. Client có thể là publisher, consumer hoặc là admin.

Việc đầu tiên bạn cần phải làm đó là khởi động Kafka server trên 1 chiếc máy tính.

## Khởi động Kafka server
1. **Bước 1**: Mở terminal lên vào nhập dòng lệnh sau để tải Kafka:
```bash
cd ~/Downloads
wget https://downloads.apache.org/kafka/2.7.0/kafka_2.13-2.7.0.tgz
```
2. **Bước 2**: Giải nén file vừa tải về:
```bash
tar -xzf kafka_2.13-2.7.0.tgz
cd kafka_2.13-2.7.0
```
3. **Bước 3**: Khởi động ZooKeeper service:
```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
```
4. **Bước 4**: Mở 1 terminal mới và nhập dòng lệnh sau đây để bắt đầu chạy Kafka server:
```bash
bin/kafka-server-start.sh config/server.properties
```

## Cơ chế hoạt động
Trước khi bắt tay vào viết ứng dụng này, mình sẽ nói 1 chút về cách nó hoạt động như sau:

- Mỗi user khi được tạo sẽ được cấp cho 1 định danh duy nhất, ở đây mình dùng tên của user cho đơn giản. Mỗi user cũng sẽ được cấp 1 topic duy nhất mang tên của chính user đó. Ví dụ: user có tên là user1 sẽ được cấp cho topic cũng tên là user1.
- Mỗi user luôn subscribe vào topic của chính họ, và publish vào topic của các user khác.
- Giả sử user A và user B đang chat với nhau, user A muốn gửi tin nhắn cho B thì sẽ publish nội dung tin nhắn vào topic B, sau đó user B đang subscribe trên topic của mình sẽ nhận được tin nhắn từ A, quá trình nhắn tin từ B tới A cũng diễn ra tương tụ.

![Minh họa cách thức giao tiếp giữa 2 user A và B](kafka-message.svg)

## Code nào

Để có thể giao tiếp được với Kafka server, bạn cần dùng thư viện kafka-python. Sử dụng pip để cài đặt thư viện này:
```bash
pip install kafka-python
```
Bạn có thể đọc document của thư viện này tại [đây](https://kafka-python.readthedocs.io/en/master/usage.html).

Trước tiên, mình thử publish vào 1 topic bằng đoạn code sau:
```python publisher.py
from kafka import KafkaProducer

BOOTSTRAP_SERVERS = ['localhost:9092']
producer = KafkaProducer(bootstrap_servers=BOOTSTRAP_SERVERS,
                         value_serializer=lambda value: bytes(value, encoding='utf-8'))

username1 = input('What is your name? ')
username2 = input('Who do want connect? ')

while True:
    value = input()
    producer.send(username2, value=value)
    producer.flush()
```

Đoạn code này sẽ hỏi tên của bạn và tên của user mà bạn muốn nhắn tin, sau đó nó bắt đầu 1 vòng lặp forever để chờ bạn nhập tin nhắn và gửi tin nhắn đó đến topic của user kia.

`BOOTSTRAP_SERVERS` chứa thông tin để kết nối đến server Kafka mà mình vừa khởi động ở trên, vì mình chạy Kafka trên máy của mình nên địa chỉ IP của server Kafka là localhost và port mặc định là 9092.

Một message trong Kafka có 2 trường: `key` và `value`. Bạn có thể truyền bất cứ dữ liệu nào vào trong 2 trường này. Ở đây mình chỉ cần gửi nội dung tin nhắn nên dùng trường `value` là đủ.

Dữ liệu trong 1 message phải có dạng `byte`. Vậy nên trước khi gửi mình phải chuyển tin nhắn từ dạng `string` sang `byte` bằng cách truyền tham số `value_serializer` cho 1 hàm lambda.

Test script này bằng lệnh trong terminal:
```bash
python publisher.py
```
![Chạy thử script chat.py với tên là user1 và user2](Screenshot-2021-01-01-160825.png)

Vì mình mới chỉ chạy ở bên phía user1 nên không có bất cứ phản hồi nào từ user2. Giờ mình sẽ viết code để user2 lắng nghe tin nhắn từ user1:
```python subscriber.py
from kafka import KafkaConsumer

BOOTSTRAP_SERVERS = ['localhost:9092']
username2 = input('What is your name? ')

consumer = KafkaConsumer(username2, auto_offset_reset='latest',
                         bootstrap_servers=BOOTSTRAP_SERVERS,
                         value_deserializer=lambda value: value.decode('utf-8'))

for msg in consumer:
    print(msg.value)
```
Đoạn code trên khởi tạo 1 instance `KafkaConsumer` để subscribe vào topic của user2 và bắt đầu 1 vòng lặp for để lắng nghe trên topic này. Vòng lặp for này sẽ chờ message từ topic trong thời gian vô hạn và không bao giờ dừng lại trừ khi bạn dừng chương trình.

Tham số `value_deserializer` là 1 hàm được dùng khi consumer nhận được message từ topic để chuyển trường `value` của message từ dạng `byte` trở về dạng ban đầu của nó, trong trường hợp này là `string`.

Để chạy script này, mở một tab terminal mới và chạy lệnh:
```bash
python subscriber.py
```

Nhập tên user2 để bắt đầu lắng nghe.

Quay lại tab đang chạy publisher, thử nhập 1 message bất kỳ và quay lại tab đang chạy subscriber. Bạn sẽ thấy message mà mình vừa nhập hiện trên terminal của subscriber.

![Chạy thử script chat.py với tên là user1 và user2](Screenshot-2021-01-01-161638.png)

Như vậy là chúng ta đã thiết lập được giao tiếp 1 chiều giữa 2 user. Tiếp theo, mình sẽ làm cho mỗi user vừa publish vừa subscribe vào Kafka để có thể vừa gửi và nhập tin nhắn.

Như bạn thấy trong 2 đoạn code trên, cả 2 đều chứa 1 vòng lặp vô hạn. Điều này có nghĩa là khi user đang gửi tin nhắn thì không thể nhận tin nhắn và ngược lại, vì bản chất của 2 đoạn code trên là blocking - chương trình sẽ chờ trong vô hạn để hoàn thành 1 tác vụ rồi mới chuyển sang tác vụ kế tiếp.

Để giải quyết vấn đề này, mình sẽ sử dụng thread để tạo 1 thread chạy song song với thread chính. Thread chính có nhiệm vụ là gửi tin nhắn cho đối phương còn thread được tạo có nhiệm vụ nhận tin nhắn từ đối phương.

Trong Python, để làm việc với thread, mình sử dụng thư viện threading có sẵn. Cách sử dụng khá đơn giản, chỉ cần khởi tạo 1 instance `Thread` với 2 tham số: `target` là hàm sẽ chạy trong thread mới; và `args` là tham số truyền vào hàm `target`. Sau đó gọi method `start`. Ví dụ:

```python
thread = threading.Thread(target=foo, args=())
thread.start()
```

Vậy trước khi bắt đầu vòng lặp while trong publisher, mình chỉ cần tạo 1 thread mới chạy hàm của subscriber là xong. Dưới đây là đoạn code hoàn chỉnh của chương trình:

```python chat.py
import threading

from kafka import KafkaConsumer, KafkaProducer

BOOTSTRAP_SERVERS = ['localhost:9092']

def subscribe(topic):
    consumer = KafkaConsumer(topic, auto_offset_reset='latest',
                             bootstrap_servers=BOOTSTRAP_SERVERS,
                             value_deserializer=lambda value: value.decode('utf-8'))
    for msg in consumer:
        print(msg.value)

username1 = input('What is your name? ')
username2 = input('Who do want connect? ')

consumer_thread = threading.Thread(target=subscribe, args=(username2,))
consumer_thread.start()

producer = KafkaProducer(bootstrap_servers=BOOTSTRAP_SERVERS,
                         value_serializer=lambda value: bytes(value, encoding='utf-8'))

while True:
    key = username1
    value = input()
    producer.send(username1, value=value)
    producer.flush()
```

Bạn thử mở 2 tab terminal, cho mỗi terminal chạy script này. Tab 1 nhập lần lượt user1 và user2, tab 2 nhập ngược lại user2 và user1.

Khi bạn nhập 1 tin nhắn ở tab 1 và nhấn Enter thì tin nhắn đó sẽ được hiện trên tab 2, điều tương tự cũng diễn ra khi bạn nhập vào tab 2.

Bạn có thể xem code đầy đủ của bài viết này ở [đây](https://github.com/duydvu/kafka-tutorial).

Như vậy là chúng ta đã hoàn thành xong 1 ứng dụng đơn giản để chat trên console bằng Python và Kafka. Mặc dù vậy, nó vần còn thiếu nhiều tính năng quan trọng như:
- Chỉ cho 2 user nhắn tin với nhau trong 1 phiên riêng biệt. Hiện tại, nếu có nhiều hơn 2 user thì sẽ có trường hợp 1 user cùng nhắn tin với 2 user khác.
- Chỉ cho phép user này nhắn tin với user kia khi được họ cho phép.

Mình sẽ thực hiện những tính năng này trong phần 2 của series.

Happy Coding!
