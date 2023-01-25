---
title: Cách thiết lập môi trường Docker cho server Jupyter của bạn
date: 2021-01-05 10:08:36
index_img: /images/venti-views-1cqIcrWFQBI-unsplash.jpg
banner_img: /images/venti-views-1cqIcrWFQBI-unsplash.jpg
tags:
- Docker
- Jupyter Notebook
- Jupyter Lab
---
Jupyter Notebook và Jupyter Lab là 2 môi trường tuyệt vời cho data scientist thực hành với dữ liệu của mình. Tuy nhiên, đôi khi việc thực hành với dữ liệu gặp 1 chút khó khăn. Nào là cài đặt Java để chạy VnCoreNLP, cài tensorflow GPU để huấn luyện model, cài cmake để cài CocCoc Tokenizer, quản lý dependency,... Có khi bạn lỡ làm sai gì đó khiến cho máy bạn gặp vấn đề mà không biết giải quyết làm sao, vừa tốn công sức vừa tốn thời gian cho những tác vụ không liên quan. Docker chính là chìa khóa để giái quyết những vấn đề trên. Bài viết này sẽ hướng dẫn bạn cách thiết lập môi trường Docker để chạy server Jupyter của bạn.

<escape><!-- more --></escape>

## Docker là gì?

Nếu bạn chưa biết về Docker hoặc biết 1 chút và muốn tìm hiểu thêm thì nên đọc phần này, nếu không thì bạn có thể kéo xuống [phần tiếp theo](#Tao-Docker-image).

Docker là 1 môi trường ảo hóa giống như VirtualBox hay VMware. Nếu bạn đã từng dùng máy ảo để chạy 1 hệ điều hành thì có thể thấy rằng chúng cung cấp cho bạn 1 môi trường độc lập với máy chính, những gì bạn làm trên máy ảo không ảnh hưởng đến máy chính. Nếu bạn lỡ phá gì đó hay dính malware thì chỉ cần reset máy ảo đó đi là xong. Sự tiện lợi này khiến cho máy ảo rất thích hợp cho việc phát triển phần mềm, triển khai phần mềm, test virus,... Nhưng cũng chính sự tiện lợi này khiến cho nó tốn rất nhiều tài nguyên của máy. Vì vậy, việc sử dụng máy ảo tương đối khó khăn.

Docker ra đời nhằm khắc phục những hạn chế trên của máy ảo. Đối với máy ảo, nó dùng 1 thành phần gọi là hypervisor để mô phỏng phần cứng của nhiều máy trên 1 máy, nhờ đó cho phép nhiều máy ảo chạy trên 1 máy chính duy nhất, mỗi máy ảo có thể chạy hệ điều hành khác nhau. Trong khi đó, Docker tận dụng hệ điều hành của máy chính để chạy các container trên đó, mỗi container là 1 phần mềm được đóng gói chạy hoàn toàn độc lập với nhau. Bạn có thể làm gần như bất cứ điều gì trên container này mà không ảnh hướng đến container khác. Hình dưới đây sẽ giải thích rõ hơn về sự khác nhau giữa 2 kiến trúc này.

![So sánh kiến trúc giữa Docker và máy ảo](docker.png)

Nhờ vào việc tận dụng tài nguyên có sẵn trong hệ điều hành của máy chính, Docker tiêu tốn ít RAM và dung lượng đĩa hơn so với máy ảo. Qua đó tạo điều kiện thuận lợi cho các nhà phát triển sử dụng Docker để việc phát triển phần mềm trở nên dễ dàng hơn, tình trạng code chạy ổn trên máy này nhưng bị bug trên máy kia được giảm thiểu vì giờ đây code đều chạy trên cùng 1 môi trường. Triển khai phần mềm cũng dễ dàng hơn, trước đây thì DevOps phải biết về các dependency của phần mềm để cài đặt trước khi triển khai, giờ thì chỉ cần Developer tạo 1 image chứa phần mềm và tất cả dependency rồi đưa nó cho DevOps, tới lượt DevOps thì chỉ cần `run` nó là xong.

Đối với Data Scientist, việc sử dụng Docker cũng giúp ích rất nhiều. Ví dụ bạn có 1 tool yêu cầu Python 3.7 để chạy nhưng bạn lại đang dùng Python 3.5 thì bạn không cần phải nâng cấp Python của máy mà chỉ cần chạy tool đó trong 1 container có Python 3.7 là xong. Hoặc là trong quá trình cài đặt các phần mềm trên máy, đôi lúc bạn sẽ gặp trường hợp lỗi do cấu hình sai gì đó, nếu mà bạn không tìm được cách sửa thì sẽ tiêu tốn khá nhiều thời gian. Ngược lại, khi bạn cài trên Docker thì chỉ cần xóa container đang chạy rồi khởi động lại cái khác là xong. Mọi thứ rất nhanh phải không nào!

Docker còn mang lại nhiều lợi ích hơn trong 1 team R&D có nhiều Data Scientist. Bạn không cần phải quan tâm đến việc cài đặt hay cập nhật package này có ảnh hưởng đến thành viên khác trong team hay không, vì mọi thứ đã được **isolated** trong 1 container. Có thể bạn sẽ nghĩ rằng virtualenv của Python cho phép làm điều tương tự, nhưng Docker cho phép làm nhiều hơn thế không chỉ có mỗi quản lý Python package. Giả sử như bạn muốn dùng Java phiên bản mới hơn để chạy tool này nhưng đồng nghiệp của bạn lại đang dùng phiên bản cũ hơn không tương thích thì sao?

Tóm lại, Docker là 1 công cụ tuyệt vời đã, đang và sẽ ngày càng phát triển trong tương lai, đóng một vai trò quan trọng trong việc phát triển phần mềm của thế giới. Vì vậy, nếu bạn chưa rành về Docker thì mình khuyên bạn nên bắt đầu tìm hiểu nó để cảm nhận rõ được những lợi ích của nó nhé.

Nguồn tài liệu để học Docker:
- [Docker Documentation](https://docs.docker.com/get-started/overview/)
- [Video hướng dẫn về Docker rất bổ ích của anh Phạm Huy Hoàng](https://www.youtube.com/watch?v=1k8pox8mkxc)

Trước khi đi vào phần kế tiếp, bạn hãy chắc rằng mình đã nắm được những kiến thức cơ bản của Docker như image, container là gì rồi hãy bắt đầu nhé.

## Tạo Docker image

OK. Trong phần này mình sẽ hướng dẫn cách tạo 1 image dùng để chạy server Jupyter.

Trong thư mục trống bất kỳ, tạo 1 file mới và đặt tên là `Dockerfile`, lưu ý là không được có đuôi extension. Mở file này bằng 1 trình editor và copy nội dung sau vào file:

```docker
# Có thể chọn image khác ở https://hub.docker.com/_/python
FROM python:3.7

RUN pip install jupyterlab
WORKDIR /code
CMD ["jupyter-lab", "--ip", "0.0.0.0", "--allow-root"]
```

Sau đó, chạy lệnh trong terminal:
```bash
docker build -t jupyter-server .
```

Lệnh này sẽ build 1 image chứa Python 3.7 và Jupyter trong đó.

Sau khi build xong image, bây giờ bạn có thể sử dụng nó để chạy server Jupyter rồi. Để chạy server, chỉ cần chạy lệnh:
```bash
docker run -it -p 8888:8888 jupyter-server
```

Bạn sẽ thấy thông báo server Jupyter đã được khởi động kèm theo token. Mở browser truy cập vào địa chỉ [http://localhost:8888](http://localhost:8888) rồi nhập token đó vào, thế là bạn đã chạy được server Jupyter trên Docker rồi.

**Lưu ý**: Tất cả các file bạn tạo trên server Jupyter này đều được nằm trong container, bạn sẽ không tìm thấy thư mục code của bạn trong hệ thống file của máy chính. **Nếu bạn có lỡ xóa container này thì mọi nội dung bạn tạo trong nó cũng bị xóa theo**. Để tránh điều này, bạn nên mount thư mục `/code` trong container lên 1 thư mục trong hệ thống file của máy chính. Để làm điều vậy, bạn chỉ cần thêm 1 tham số trước khi chạy container:
```bash
docker run -it -p 8888:8888 -v <path-to-your-directory>:/code jupyter-server
```
Thay thế `<path-to-your-directory>` thành đường dần đến thư mục mà bạn muốn để code ở đó. Tham số vừa thêm vào có tác dụng mount thư mục trên máy chính của bạn lên thư mục `/code` trong container. Hiểu đơn giản là từ bây giờ 2 thư mục này có thể đồng bộ với nhau, nếu bạn tạo 1 file trong thư mục này thì nó sẽ xuất hiện trong thư mục còn lại. Nếu container bị xóa thì nội dung trong thư mục bạn chọn để mount vào vẫn y như cũ.

## Cài đặt thư viện

Bạn có thể cài thêm các thư viện cần thiết vào trong container đang chạy bằng 1 trong 3 cách sau đây

### Cách 1: Lệnh docker exec

Để dùng lệnh này, trước tiên bạn cần phải biết tên hoặc ID của container. Mở 1 terminal mới và nhập lệnh sau:
```bash
docker ps
```

Lệnh này sẽ in ra danh sách những container đang chạy trong Docker server, nội dung tương tự như sau:
```text
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
b3e70a2ab109        jupyter-server      "jupyter-lab --ip 0.…"   8 minutes ago       Up 7 minutes        0.0.0.0:8888->8888/tcp   nifty_liskov
```

Cột `CONTAINER ID` hiện ID của container của bạn, còn cột `NAMES` hiện tên của container. Mỗi container khi được tạo sẽ được cấp cho 1 ID và tên duy nhất trong số các container đang còn tồn tại trong Docker server. Copy 1 trong 2 field này từ màn hình terminal của bạn và nhập lệnh dưới đây:
```bash
docker exec -it <paste-here> bash
```
Sau khi nhấn Enter, terminal của bạn sẽ trở thành terminal trong container. Giờ bạn có thể cài bất cứ thư viện nào trong đây. Ví dụ như:
```bash
pip install numpy pandas
```

### Cách 2: Dùng tính năng shell command trên Jupyter

Jupyter cho phép ta chạy bất cứ lệnh nào trong notebook.

Để làm vậy, bạn cần phải thêm dấu chấm thang (!) vào trước lệnh shell rồi chạy notebook. Ví dụ:
```bash
!pip install numpy
```
Hình dưới đây là kết quả sau khi chạy lệnh trên trong notebook

![Chạy lệnh shell command trong notebook](screenshot.png)

Cách này nhanh hơn rất nhiều so với cách 1.

### Cách 3: Dùng terminal của Jupyter Lab

Trên thanh công cụ của Jupyter Lab, chọn **File** → **New** → **Terminal**. Một tab mới được mở ra và bạn có thể nhập lệnh vào terminal để cài thư viện.

Cách làm này chậm hơn cách thứ 2 nhưng mà phù hợp với những tác vụ như monitor bằng lệnh `top` hoặc `watch` hoặc `tail`.

## Tổng kết

Như vậy là bạn đã có 1 Docker container chạy server Jupyter để phục vụ cho công việc Data Science. Trong container này, bạn có thể cài đặt bất cứ phần mềm nào mà không sợ ảnh hưởng đến hệ điều hành trên máy bạn. Mỗi khi muốn tắt server, bạn có thể vào terminal dùng để chạy server và nhấn **Ctrl + C** là xong.

Ngoài việc isolate nội dung thư mục, Docker còn cho phép bạn giới hạn tài nguyên cho container theo ý thích. Bạn có thể giới hạn số CPU mà container được sử dụng, hoặc lượng RAM tối đa được dùng để tránh trường hợp hết RAM gây đứng máy. Tìm hiểu ở [đây](https://docs.docker.com/config/containers/resource_constraints/) để xem cách làm.

Còn 1 vấn đề mà mình chưa giải quyết là làm sao để sử dụng GPU trong Docker, nếu bạn đang làm dự án liên quan đến Deep Learning thì GPU cực kỳ cần thiết để tăng tốc quá trình huấn luyện mô hình. Vì vậy, mình sẽ giải quyết vần đề này trong bài viết theo.

Happy Coding!
