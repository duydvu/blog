---
title: Cấu hình GPU trong Docker
date: 2021-01-10 11:29:40
tags:
- Docker
- Jupyter
---
Trong bài [trước](/2021/01/05/Cach-thiet-lap-moi-truong-Docker-cho-server-Jupyter-cua-ban), mình đã trình bày cách thiết lập môi trường Docker để chạy server Jupyter. Với nó, bạn có thể cài đặt bất kỳ package nào mà bạn muốn như scikit-learn, Tensorflow, PyTorch. Nhưng những thư viện này sẽ không thể sử dụng được GPU trong máy bạn như khi cài trực tiếp trên hệ điều hành bởi vì bạn chưa cấu hình GPU cho container của bạn. Trong bài này, mình sẽ hướng dẫn bạn cách để sử dụng được GPU trong Docker.

Mình giả định rằng máy của bạn đang dùng Nvidia GPU và bạn đã cài driver cần thiết trên hệ điều hành Ubuntu rồi. Nếu bạn chưa cài đặt driver cho GPU thì vào đường link ở [đây](https://www.nvidia.com/Download/index.aspx), chọn tải rồi cài driver trước nhé.

<escape><!-- more --></escape>

## Cài đặt nvidia-container-runtime
Mở terminal và chạy đoạn lệnh dưới đây:
```bash
curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list
sudo apt-get update
```

Sau đó, nhập lệnh sau để cài đặt nvidia-container-runtime:

```bash
sudo apt-get install nvidia-container-runtime
```

Khởi động lại Docker server:
```bash
sudo service docker restart
```

Thế là xong. Đến bước tiếp theo là khởi động 1 container mới với cấu hình cho phép truy cập GPU.

## Khởi động Docker container

Để sử dụng GPU trong 1 container, bạn phải thêm tham số `--gpus` trong lệnh chạy `docker run`. Ví dụ:
```bash
docker run -it -p 8888:8888 --gpus all jupyter-server
```

Tham số `--gpus` có giá trị là `all` ở đây nghĩa là bạn cho phép container này truy cập vào toàn bộ GPU mà máy bạn đang có. Hoặc là bạn có thể chỉ định những GPU cụ thể được sử dụng trong container.

Mỗi GPU trong máy được đánh giá trị index bắt đầu từ 0. Ví dụ nếu bạn có 2 GPU thì chúng sẽ được đánh index lần lượt là 0 và 1. Để xem index của từng GPU và các thông số khác như memory, usage thì có thể dùng lệnh `nvidia-smi`. Ví dụ như trong hình dưới đây:

![Ví dụ chạy lệnh nvidia-smi](Screenshot-2021-01-10-121331.png)

Ở cột ngoài cùng bên trái, bạn sẽ thấy index của từng GPU. Ở đây có 2 GPU nên được đánh lần lượt 0 và 1. GPU 0 đang sử dụng 4905 MB / 11178 MB, còn GPU 1 thì đang dùng 11140 MB / 11178 MB. Nếu bạn chỉ muốn cấp cho container sử dụng GPU 0 thì dùng lệnh như sau:
```bash
docker run -it -p 8888:8888 --gpus device=0 jupyter-server
```

Hoặc nếu bạn có nhiều hơn 2 GPU thì bạn có thể chỉ định bất kỳ GPU nào được dùng trong container như sau:
```bash
docker run -it -p 8888:8888 --gpus device=0,2,4 jupyter-server
```

## Tổng kết

Giờ đây bạn có thể cài thư viện Tensorflow GPU hay PyTorch trong server Jupyter là có thể dùng được ngay GPU. Với Docker, bạn có thể tha hồ cài đặt bất kỳ phần mềm nào mà bạn muốn.

Mình cũng chia sẻ thêm một số package rất hữu ích mà mình hay dùng trong công việc:
- **tmux**: cho phép bạn quản lý terminal theo phong cách xịn xò như 1 hacker, bạn có thể chạy 1 chương trình trong terminal và tắt nó đi mà không sợ bị dừng chương trình.
- [**ohmyzsh**](https://github.com/ohmyzsh/ohmyzsh): terminal đẹp lung linh kèm theo 1 số plugin như git cho phép bạn thao tác trên terminal tiện lợi và thích thú hơn.

Happy Coding!