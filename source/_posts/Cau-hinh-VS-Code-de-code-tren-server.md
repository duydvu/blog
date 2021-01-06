---
title: Cấu hình VS Code để lập trình trên server bằng Remote - SSH
date: 2020-12-26 11:18:29
photos:
- /images/florian-krumm-yLDabpoCL3s-unsplash.jpg
tags:
- IDE
- Coding
- VScode
---
Trong bài viết này, mình sẽ hướng dẫn cho các bạn cách thiết lập môi trường lập trình trên server hơn thông qua extension Remote - SSH của VS Code.

Trước khi bắt đầu, mình sẽ trả lời cho câu hỏi:

## Tại sao phải lập trình trên server?

Đối với đa số lập trình viên, 1 chiếc laptop hoặc PC là đủ để phục vụ cho công việc của họ. Tuy nhiên, đối với Data Scientist thì đôi khi làm việc trên 1 thiết bị là chưa đủ. Bạn cần phải làm việc trên máy tính có cấu hình mạnh, bộ nhớ lớn để có thể load dữ liệu và huấn luyện mô hình, đặc biệt là các mô hình Deep Learning yêu cầu phải có 1 hoặc nhiều GPU và thời gian huấn luyện có khi lên tới hàng tuần. Những chiếc máy tính mạnh mẽ như vậy thường sẽ không vừa vặn với 1 chiếc laptop hoặc PC nên nếu làm việc trực tiếp thì sẽ khá khó khăn. Vì vậy, việc lập trình trên server sẽ giúp ích rất nhiều, ví dụ như:
- Bạn có thể huấn luyện mô hình mà vẫn có thể tắt laptop mang đi nơi khác.
- Bạn có thể ra quán coffee với chiếc máy tính mỏng nhẹ mà vẫn làm việc được trên chiếc máy tính khủng.
- Bạn có thể chia sẻ máy với đồng nghiệp để cùng tận dụng tài nguyên của máy.

Hơn nữa, các dịch vụ điện toán đám mây như là AWS, GCP cung cấp dịch vụ tính toán giúp cho chúng ta có thể làm việc trên những chiếc máy tính mạnh mà không cần phải bỏ 1 số tiền lớn để mua hoặc thuê về, điều này cũng khiến cho việc lập trình trên server ngày càng trở nên phổ biến.

Có nhiều cách khác nhau để có thể lập trình được trên server, cách nhanh nhất là bật 1 trình soạn thảo văn bản như Vim hoặc Nano rồi bắt đầu code. Tuy nhiên, những trình soạn thảo này không cung cấp sẵn những tính năng hữu ích của 1 IDE như là syntax highlighting, autocompletion, debugger. Vim là 1 trình soạn thảo rất tuyệt vời vì nó cho phép gắn thêm plugin vào để thêm tính năng, nó cho phép bạn biến nó thành 1 IDE thực thụ thông qua việc cài đặt những plugin được cung cấp bởi rất nhiều lập trình viên khác. Tùy vào sở thích mà bạn có thể chọn Vim là công cụ chính cho việc lập trình trên server, mặc dù việc thao tác trên Vim khá phức tạp nhưng khi bạn đã quen với việc sử dụng nó rồi thì bạn sẽ thấy được sự mạnh mẽ và hiệu quả mà nó đem lại. Mình cũng hay sử dụng Vim cho những tác vụ edit đơn giản trên server vì nó rất nhanh và tiện lợi. Tạm gác Vim qua một bên, trong bài này mình sẽ chỉ tập trung vào cách cấu hình để bạn có thể lập trình trên server thông qua trình soạn thảo VS Code nổi tiếng của Microsoft (nếu bạn đã từng dùng Visual Studio rồi thì nên tránh nhầm lẫn nhé vì đây là 2 phần mềm khác nhau, Visual Studio là 1 IDE còn VS Code chỉ là 1 trình soạn thảo văn bản mà thôi!).

Bắt đầu nào!

## Cài đặt extension Remote - SSH

Mình giả định rằng bạn đã cài đặt sẵn VS Code trong máy rồi nên mình sẽ chỉ hướng dẫn sau khi bạn đã bật VS Code lên nhé.

Để cài đặt extension trên VS Code, bạn hãy nhấn vào biểu tượng **Extensions** ở thanh công cụ bên trái, sau đó, trên thanh tìm kiếm gõ cụm từ *"Remote - SSH"*.

Extension mà ta cần tìm sẽ xuất hiện trong kết quả tìm kiếm, nhấn vào đó rồi nhấn nút **Install** để bắt đầu quá trình cài đặt. Sau khi cài đặt xong, khởi động lại VS Code để load extension lên.

![Giao diện VS Code sau khi hoàn tất cài đặt Remote - SSH](Screenshot-2020-12-26-120808.png)

Tiếp theo, mình sẽ hướng dẫn cách cấu hình môi trường lập trình trên server.

## Cấu hình môi trường lập trình trên server

### Bước 1: Tạo SSH key

Trước khi kết nối đến server thông qua VS Code, bạn cần phải tạo 1 SSH key để xác thực quyền kết nối của máy bạn với server mà không cần dùng mật khẩu.
Nếu bạn đã cấu hình SSH server bằng SSH key rồi thì hãy đến với [bước 3](#Buoc-3-Thiet-lap-moi-truong-SSH-tren-VS-Code).

Mở 1 terminal mới và nhập lệnh sau:
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
```
Câu lệnh trên sẽ trả về kết quả như dưới đây:
```
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
```
Nhấn Enter 2 lần để hoàn tất việc tạo SSH key trên máy tính của bạn, key này được lưu tại `~/.ssh/id_rsa`.

### Bước 2: Thêm SSH key vào danh sách authorized keys của server

Tiếp theo, bạn cần phải SSH vào server để thêm SSH key mà bạn vừa tạo vào danh sách authorized keys của server. Bước này dùng để xác thực quyền truy cập vào server cho máy tính của bạn.

Đầu tiên, bạn cần copy public key sử dụng lệnh dưới đây:
```bash
cat ~/.ssh/id_rsa.pub
```
Copy toàn bộ nội dung trả về của lệnh này.

Tiếp theo, kết nối SSH vào server bằng lệnh sau:
```bash
ssh <username>@<ip-address>
```
Thay `username` và `ip-address` thành giá trị tương ứng mà bạn có. Nhập mật khẩu để xác thực.

Sau đó, chèn nội dung của public key vào cuối file `~/.ssh/authorized_keys` bằng lệnh sau:
```bash
echo "<paste-your-public-key-content-here>" >> ~/.ssh/authorized_keys
```
Thay `paste-your-public-key-content-here` bằng nội dung của public key mà bạn vừa copy.

Như vậy là bạn đã hoàn tất việc xác thực SSH key với server, trong những lần ssh sau, bạn không cần phải cung cấp mật khẩu nữa giúp cho việc ssh vào server trở nên thuận tiện hơn.

**Lưu ý**: Nếu ở trong bước 1, bạn sử dụng tên khác để tạo SSH key như `id_abc` thì bạn cần phải làm thêm 1 bước là thay đổi cấu hình SSH để nó biết được cần phải dùng key nào khi kết nối đến server của bạn.

Trong file `~/.ssh/config`, bạn cần thêm đoạn dưới đây vào cuối file:
```. Chỉ thêm khi file key của bạn khác với mặc định (id_rsa, id_dsa, id_ecdsa, id_ed25519)
Host <ip-address>
  HostName <ip-address>
  User <username>
  Port 22
  IdentityFile ~/.ssh/id_abc
```

### Bước 3: Thiết lập môi trường SSH trên VS Code
Sau khi đã có thể kết nối đến server thông qua SSH key, bạn hãy mở VS Code và làm theo các bước sau đây:
1. Nhấn tổ hợp phím **Ctrl + Shift + P** để mở khung **Command Palette**.
2. Nhập *"Remote-SSH: Add New SSH Host"* rồi nhấn **Enter**.
3. Nhập câu lệnh SSH mà bạn sử dụng để SSH đến server: `ssh <username>@<ip-address>`
4. Nhập đường dẫn đến file config: `~/.ssh/config`

VS Code sẽ hiện thống báo rằng server đã được thêm vào. Giờ bạn chọn biểu tượng Remote Explorer ở thanh công cụ bên trái, bạn sẽ thấy địa chỉ ip của server vừa thêm vào. Từ bây giờ, bạn có thể mở 1 folder bất kỳ trên server bằng cách nhấn nút **Connect to Host in New Window** ở kế bên địa chỉ ip của server.

Như vậy là bạn có thể bắt đầu lập trình trên server thông qua VS Code rồi đó.

Một lưu ý nhỏ là khi dùng VS Code trên server thì bạn cần phải cài extension **trên server đó** thì mới có thể dùng được, tức là nếu có extension nào bạn đã cài trên máy rồi thì vẫn phải cài lại trên server.

Happy coding!
