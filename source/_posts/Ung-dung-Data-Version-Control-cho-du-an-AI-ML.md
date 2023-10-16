---
title: "Giới thiệu về Data Version Control dùng cho các dự án AI/ML"
date: 2023-10-16 21:00:00
excerpt: Ứng dụng DVC giúp cho việc quản lý dữ liệu và mô hình trở nên dễ dàng và hiệu quả hơn, nhất là những dự án có quy mô lớn và nhiều người tham gia.
index_img: /images/brandon-morgan-3qucB7U2l7I-unsplash-small.jpg
banner_img: /images/brandon-morgan-3qucB7U2l7I-unsplash-medium.jpg
tags:
- Data Science
- Data Version Control
- Tool
- Machine Learning
- Artificial Intelligence
---

Một trong những vấn đề quan trọng nhất trong các dự án AI/ML là quản lý dữ liệu và mô hình. Các dự án AI/ML thường có nhiều người tham gia và thường xuyên phải thay đổi. Những thay đổi có thể là thay đổi về dữ liệu, về kiến trúc mô hình, thay đổi tham số, thay đổi cách thức huấn luyện, thay đổi cách thức đánh giá mô hình, thay đổi cách thức triển khai mô hình, v.v...

Nếu chúng ta không quản lý dữ liệu và mô hình một cách hiệu quả thì sẽ dễ dẫn đến nhiều tình huống không mong muốn. Ví dụ như:
- Tình huống 1:
  - Bạn train một mô hình với một tập dữ liệu.
  - Bạn chỉnh sửa tập dữ liệu và train lại mô hình nhưng không lưu lại phiên bản cũ.
  - Khi đánh giá mô hình, bạn nhận thấy mô hình mới không tốt bằng mô hình cũ.
  - Bạn muốn quay lại phiên bản tập dữ liệu cũ để tìm ra nguyên nhân nhưng không thể vì bạn đã xóa nó đi rồi 😰
  - Tất nhiên là việc lưu 1 bản backup mỗi khi chỉnh sửa tập dữ liệu sẽ giúp tránh được tình huống này. Nhưng khi số lần chỉnh sửa nhiều lên khiến cho số bản backup càng nhiều, bạn sẽ rất khó quản lý chúng.
- Tình huống 2: Bạn vừa được tham gia vào 1 dự án đang trong quá trình phát triển. Dự án này trước đây được phụ trách bởi 1 anh đồng nghiệp trong team. Bạn nhận code & data từ ảnh về và bắt đầu công việc.
  - Bạn: "Em đã thay đổi tham số và train lại model. Model mới có độ chính xác cao hơn model cũ mà anh train cách đây 3 tháng."
  - Anh đồng nghiệp: "Tuyệt vời! Để anh đánh giá model của em trước khi deploy nó lên prod nhé."
  - Anh đồng nghiệp: "Hmmm... Model của em có vẻ không chính xác bằng model của anh. Model của em là 96% trong khi của anh là 98%."
  - Bạn: "Sao lại thế ạ? Trong report của anh về model được train 3 tháng trước thì độ chính xác chỉ là 94% thôi mà!"
  - Anh đồng nghiệp: "Lạ nhỉ? À anh nhớ rồi. Trước khi em join project này cách đây 1 tuần thì anh có làm sạch tập test nên khiến cho model chính xác hơn. Chưa kịp update report thì em đã train lại rồi. 😅"
  - Bạn: "..."
  
Ở đây, việc không tracking thay đổi của tập dữ liệu khiến cho bạn không biết được rằng model trước đây được đánh giá trên tập dữ liệu như thế nào.

Trước đây, mình đã từng đặt tên cho các tập dữ liệu và mô hình với những cái tên như: model_v1, model_v2, model_new, data_raw, data_cleaned, data_preprocessed, v.v... Rồi 1 thời gian sau, mình lại quên mình đã làm gì với những tập dữ liệu và mô hình này. model_v1 train bằng tập dữ liệu nào? learning rate là bao nhiêu? Rồi cả tỷ câu hỏi khác mà bây giờ chỉ có cách là dùng máy du hành thời gian của Doraemon mới có thể trả lời được 😂.

Qua những ví dụ trên, chắc hẳn bạn đã phần nào hình dung được tầm quan trọng của việc quản lý dữ liệu và mô hình trong các dự án AI/ML.

Để giải quyết vấn đề này, có rất nhiều công cụ được phát triển nhưng trong bài viết này, mình sẽ giới thiệu về [Data Version Control (DVC)](https://dvc.org/).

# Data Version Control là gì?

Data Version Control (DVC) là một công cụ mã nguồn mở dùng để quản lý phiên bản cho dữ liệu và mô hình. Trong phát triển phần mềm thông thường, chúng ta đã quá quen thuộc với việc sử dụng Git để quản lý phiên bản cho code. Tuy nhiên, Git không phù hợp để quản lý phiên bản cho dữ liệu và mô hình. DVC ra đời để giải quyết vấn đề này.

DVC được thiết kế để hoạt động dựa trên Git. Điều này có nghĩa là trong cùng 1 repo, bạn vừa có thể dùng Git để quản lý code, vừa có thể dùng DVC để quản lý dữ liệu và mô hình.

Nhờ hoạt động dựa trên Git, DVC tận dụng được các tính năng mạnh mẽ của Git như: branch, tag. Bạn có thể tạo nhiều branch, mỗi branch chứa 1 tập dữ liệu khác nhau. Bạn có thể tạo tag mỗi khi train xong 1 mô hình.

![Nguồn: dvc.org](dvc.png)

# Bắt đầu sử dụng DVC

Bạn có thể bắt đầu sử dụng DVC ngay bây giờ bằng việc cài đặt theo hướng dẫn ở link sau: [https://dvc.org/doc/install](https://dvc.org/doc/install).

Hoặc nếu bạn dùng pip, bạn có thể cài đặt DVC bằng lệnh sau:

```bash
pip install dvc
```

Sau khi cài đặt, trong thư mục chứa project repo của bạn, tiến hành khởi tạo project DVC bằng lệnh:

```bash
dvc init
```

# Làm quen với quản lý phiên bản dữ liệu

Khi sử dụng Git với code, thông thường bạn sẽ thay đổi code trong file rồi commit file đó vào index của Git. Cách này phù hợp với code vì các file code thường có kích thước rất nhỏ và dễ theo dõi những thay đổi về nội dung của chúng.

Khác với code, dữ liệu thường có kích thước lớn và thay đổi rất nhiều. Nếu commit file dữ liệu vào repo, nó sẽ trở nên rất nặng. Bạn cũng không thể biết được rằng file dữ liệu này đã được thay đổi như thế nào.

DVC tiếp cận vấn đề này bằng cách sử dụng 1 file meta để lưu lại thông tin về file dữ liệu. File meta này có kích thước nhỏ và được lưu trữ trong repo của Git. Khi bạn thay đổi file dữ liệu, DVC sẽ cập nhật file meta này. Khi bạn muốn lấy lại file dữ liệu, DVC sẽ dựa vào file meta này để tìm ra file dữ liệu cần lấy.

Ví dụ, bạn có 1 file CSV chứa dữ liệu training. Bạn muốn lưu file này vào repo của Git. Bạn sẽ làm như sau:

```bash
dvc add train.csv
```

Lệnh này sẽ tạo ra file `train.csv.dvc` chứa thông tin meta của file dữ liệu. Nội dung của nó có cấu trúc giống như thế này:
  
```yaml
outs:
- md5: 9da9c6d8d622f096132838c67e7685b6
  size: 298046
  hash: md5
  path: train.csv
```

Về cơ bản, lệnh `dvc add` sẽ tính toán ra giá trị hash của file dữ liệu và lưu vào file `train.csv.dvc`. Hàm hash sẽ đảm bảo rằng nếu file dữ liệu thay đổi thì giá trị hash cũng sẽ thay đổi.

Sau khi có được file `train.csv.dvc`, bạn commit file này vào repo của Git. File này đóng vai trò như là 1 con trỏ, nó sẽ trỏ tới file dữ liệu thật. File dữ liệu thật sẽ được lưu trữ trong thư mục `.dvc/cache`.

```bash
git add train.csv.dvc
git commit -m "Add train.csv"
```

Bây giờ, bạn thử edit file `train.csv` và lưu lại. Sau đó, bạn chạy lệnh sau:

```bash
dvc status
```

Lệnh `dvc status` có chức năng giống như `git status` nhưng dành cho dữ liệu. Lệnh này sẽ kiểm tra xem file dữ liệu đã được thay đổi hay chưa. Nếu file dữ liệu đã được thay đổi, lệnh này sẽ thông báo cho bạn biết:

```
train.csv.dvc:
        changed outs:
                modified:           train.csv
```

Vì file dữ liệu đã thay đổi, file `train.csv.dvc` không còn trỏ đến đúng file dữ liệu nữa. Bạn cần phải update lại file này bằng lệnh:

```bash
dvc commit train.csv.dvc
```

DVC sẽ hỏi là bạn có muốn commit file này hay không. Nhập `y` và Enter để tiếp tục.

Lúc này, DVC sẽ tính toán lại giá trị hash của file dữ liệu và cập nhật lại file `train.csv.dvc`. Sau đó, bạn commit file vào Git.

```bash
git add train.csv.dvc
git commit -m "Update train.csv"
```

Giả sử như bạn muốn lấy lại file dữ liệu cũ. Bạn có thể làm theo cách sau:

```bash
git checkout HEAD~1 -- train.csv.dvc
dvc checkout train.csv
```

Lệnh `git checkout` sẽ lấy lại file `train.csv.dvc` từ commit trước đó. Như hồi nãy mình đã nói, file meta đóng vai trò như là 1 con trỏ để trỏ tới dữ liệu thật. Lúc này, file `train.csv.dvc` không còn trỏ đến file `train.csv` trong thư mục chính nữa mà đang trỏ tới file cũ đang nằm trong `.dvc/cache`.

Khi tới câu lệnh `dvc checkout`, DVC sẽ dựa vào file meta để tìm ra file dữ liệu thật và đem file này về thư mục chính. Thế là bạn đã lấy lại được file dữ liệu cũ rồi!

Giống như Git, DVC cũng hỗ trợ lưu trữ remote nhưng thay vì là Github, Gitlab thì ta có S3, Google Cloud Storage, Google Drive, HDFS, v.v... Nhưng mình sẽ không đi sâu vào phần này, nếu muốn thì bạn có thể xem thêm tại [https://dvc.org/doc/user-guide/data-management/remote-storage](https://dvc.org/doc/user-guide/data-management/remote-storage).

Như vậy là mình đã cover được khái niệm cơ bản về quản lý phiên bản dữ liệu của DVC rồi đấy. Đối với mô hình thì về cơ bản nó cũng chỉ là 1 hoặc nhiều file có thể được `dvc add` và track bằng file `.dvc` mà thôi.

Đợi đã! Nhưng chúng ta chưa giải quyết được bài toán làm thế nào để xác định 1 mô hình được train bằng dữ liệu gì và tham số như thế nào mà, đúng không???

Việc quản lý phiên bản dữ liệu chỉ là bước khởi đầu cho quản lý phiên bản mô hình. Mình sẽ đi sâu hơn về vấn đề này trong bài viết tiếp theo. Còn bây giờ thì tạm thời dừng ở đây nhé :v.

Happy Coding!
