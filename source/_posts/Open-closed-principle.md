---
title: Open-closed principle
date: 2020-08-23 14:28:47
index_img: /images/philipp-berndt-5i0GnoTTjSE-unsplash.jpg
banner_img: /images/philipp-berndt-5i0GnoTTjSE-unsplash.jpg
tags:
- OOP
- Software Engineering
- Python
- SOLID
---

Open-closed principle là 1 trong các nguyên tắc của bộ nguyên tắc lập trình SOLID trong lập trình hướng đối tượng. Đây là 1 nguyên tắc rất quan trọng bởi nó giúp cho code của dự án dễ bảo trì và mở rộng, có thể thích ứng với những thay đổi trong môi trường Agile.

Nguyên văn phát biểu của nguyên tắc này như sau:

> "Các thực thể phần mềm (class, function,...) nên tạo điều kiện cho việc mở rộng, nhưng hạn chế cho việc thay đổi."

<escape><!-- more --></escape>

Có thể hiểu lợi ích của nguyên tắc này thông qua 1 ví dụ như sau, giả sử như bạn là một lập trình viên đang làm cho 1 công ty IT nọ. Bạn vào đó với vai trò phát triển phần mềm cho 1 dự án của 1 team trong công ty. Thật không may, mã nguồn của dự án đó nhiều đến nỗi bạn không kiểm soát được ảnh hưởng của những thành phần trong hệ thống đối với nhau. Nhưng công ty lại đòi hỏi bạn phải hiện thực 1 tính năng mới cho dự án trong 1 thời gian ngắn. Vậy làm cách nào bạn có thể hoàn thành công việc được giao đúng hạn? Bạn sợ rằng nếu bạn hiện thực tính năng mới theo yêu cầu nghĩa là bạn đang tạo ra 1 sự rủi ro khiển cho hệ thống không còn hoạt động giống như trước. Bạn liên tục đặt câu hỏi rằng liệu tính năng mới được thêm vào có làm phá vỡ những tính năng khác hiện có hay không. Nếu chuyện đó thật sự xảy ra thì nó không hẳn hoàn toàn là lỗi của bạn. Cái quan trọng nhất trong chuyện này là cách mà code của dự án đã được thiết kế như thế nào. Nếu code của dự án tuân theo open-closed principle thì bạn có thể hiện thực tính năng mới bằng cách viết thêm code cho dự án, tạo thêm class kế thừa từ class sẵn có trong dự án mà không cần phải thay đổi code có sẵn. Điều này giúp giảm thiểu rủi ro phá vỡ hệ thống đang hoạt động mà vẫn hoàn thành được tiến độ công việc của bạn. Trong khi đó, nếu dự án được thiết kế đi ngược lại open-closed principle thì bạn có thể phải thay đổi code của dự án. Rõ ràng là điều này tiềm ẩn nhiều rủi ro hơn và cũng khó hiện thực hơn trường hợp đầu tiên.

Như vậy, ta đã thấy được lợi ích của việc thiết kế code theo open-closed principle rồi. Vậy code như thế nào mới gọi là tuân theo nguyên tắc đó? Ta hãy tiếp tục tìm hiểu thông qua ví dụ Python dưới đây.

Mình đang làm 1 dự án xây dựng mô hình phân loại spam cho văn bản sử dụng Machine Learning. Để cho dễ hiểu, mình sẽ bỏ qua phần hiện thực của mô hình mà chỉ tập trung vào phần code để sử dụng mô hình mà thôi.

Giả sử như mình đã xây dựng xong 1 mô hình có input là 1 đoạn văn bản và output ra là 1 số hoặc 0 hoặc 1, 0 nghĩa là không phải spam còn 1 nghĩa là spam. Nhưng trước khi đưa nó vào chạy thực tế, mình cần phải đánh giá mô hình này có hoặc động tốt hay không bằng việc cho nó dự đoán 1 tập nhiều văn bản khác nhau và tính tỷ lệ phần trăm mà nó dự đoán đúng. Vậy mình sẽ hiện thực 1 hàm đánh giá mô hình như sau:

``` python
# Đây là hàm dùng để lấy output của mô hình
def predict(texts: List[str]) -> np.array:
    # Hàm này sẽ dùng mô hình đã được huấn luyện để chạy rồi trả về kết quả của mô hình

def evaluate_spam_model(texts: List[str], truth_labels: np.array) -> float:
    predicted_labels = predict(texts)
    num_correct_predictions = np.sum(predicted_labels == truth_labels) # Tính số lần mà mô hình dự đoán đúng
    num_total_predictions = len(texts) # Tổng số văn bản mà mô hình dự đoán.
    accuracy = num_correct_predictions / num_total_predictions # Tính accuracy của mô hình
    return accuracy
```

Khi chạy hàm này, mình nhận thấy kết quả đánh giá của mô hình không đạt yêu cầu đề ra nên mình quyết định thử dùng 1 phương pháp mới để cải thiện độ chính xác của mô hình. Sau khi hiện thực lại xong, mình nhận thấy hàm `predict` không thể chạy được mô hình mới vì bạn đã dùng thư viện khác để hiện thực nó. Bạn thay đổi code của mình thành như sau:

``` python
def predict(texts: List[str], model_name: str) -> np.array:
    if model_name == 'v1':
        # Chạy mô hình đầu tiên
    elif model_name == 'v2':
        # Chạy mô hình thứ hai
    else:
        raise Exception('No such model.')

def evaluate_spam_model(texts: List[str], truth_labels: np.array, model_name: str) -> float:
    predicted_labels = predict(texts, model_name)
    num_correct_predictions = np.sum(predicted_labels == truth_labels) # Tính số lần mà mô hình dự đoán đúng
    num_total_predictions = len(texts) # Tổng số văn bản mà mô hình dự đoán.
    accuracy = num_correct_predictions / num_total_predictions # Tính accuracy của mô hình
    return accuracy
```

Ở đây ta có 2 vấn đề:
1. Việc thêm code vào hàm `predict` làm tăng rủi ro sinh ra bug cho hàm này. Nếu như mình chỉ có 2 hoặc 3 mô hình thì không thành vấn đề. Nhưng giả sử khi mình có tới hàng chục mô hình khác nhau thì đồng nghĩa với việc hàm này sẽ vô cùng dài và khó đọc. Khi đó thì việc kiểm tra lỗi sẽ trở nên khó khăn hơn.
2. Việc thay đổi signature của hàm `predict` khiến cho những hàm sử dụng nó cũng thay đổi theo. Trong trường hợp này thì chỉ có 1 mình hàm `evaluate_spam_model` là phải thay đổi. Nhưng nếu như mình có hàm khác sử dụng hàm `evaluate_spam_model` thì sao? Ví dụ như hàm `report_spam_performance` sử dụng hàm `evaluate_spam_model` để chạy ra kết quả và output ra 1 report để mình xem. Như vậy, việc thay đổi signature của 1 hàm có thể dẫn đến việc thay đổi signature của toàn bộ hàm trực tiếp hoặc gián tiếp sử dụng nó.

Rõ ràng là đoạn code trên không tuân theo open-closed principle. Vậy code tuân theo nguyên tắc này sẽ trông như thế nào? Hãy xem đoạn code dưới đây:

``` python
class SpamStrategy(ABC): # Khai báo class kế thừa Abstract Base Classes. Link: https://docs.python.org/3/library/abc.html#abc.ABC
    @abstractmethod
    def predict(self, texts: List[str]) -> np.array:
        pass

class SpamV1Strategy(SpamStrategy):
    def predict(self, text: List[str]) -> np.array:
        # Chạy mô hình thứ nhất

class SpamV2Strategy(SpamStrategy):
    def predict(self, text: List[str]) -> np.array:
        # Chạy mô hình thứ hai


def evaluate_spam_model(texts: List[str], truth_labels: np.array, strategy: SpamStrategy):
    predicted_labels = strategy.predict(texts, model_name)
    num_correct_predictions = np.sum(predicted_labels == truth_labels) # Tính số lần mà mô hình dự đoán đúng
    num_total_predictions = len(texts) # Tổng số văn bản mà mô hình dự đoán.
    accuracy = num_correct_predictions / num_total_predictions # Tính accuracy của mô hình
    return accuracy
```

Ở đây, mình khai báo 1 abstract class `SpamStrategy` có khai báo 1 hàm abstract là `predict`. Mỗi lần hiện thực 1 mô hình spam mới thì mình sẽ tạo thêm 1 class mới kế thừa từ class này rồi chạy hàm `evaluate_spam_model` bằng việc thay đổi tham số strategy. Như vậy, mỗi lần mình hiện thực thêm 1 mô hình mới thì đoạn code của những mô hình cũ (những class kế thừa từ `SpamStrategy`) không bị thay đổi. Nếu như mình nhận thấy mô hình mới không được tốt như mô hình cũ thì mình vẫn có thể dễ dàng chạy lại mô hình cũ chỉ bằng việc thay đổi tham số của hàm `evaluate_spam_model` mà thôi!

Thông qua ví dụ trên, chúng ta thấy được tầm quan trọng của việc thiết kế code theo open-closed principle. Đối với những dự án nhỏ, nguyên tắc này đôi khi là không cần thiết bởi nó khiến cho code trở nên dài dòng hơn. Nhưng khi dự án phình to ra với vô số hàm và class được thêm vào thì ta mới thấy rõ được tầm quan trọng của nguyên tác này.