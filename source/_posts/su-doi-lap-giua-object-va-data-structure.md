---
title: Sự đối lập giữa Object và Data Structure
date: 2020-08-09 16:13:43
tags:
- OOP
- Data structure
- Software Engineering
- Java
---
Bài viết này là kiến thức mình học được khi đọc cuốn *Clean Code: A Handbook of Agile Software Craftsmanship*, mình muốn thông qua bài viết này có thể chia sẻ hiểu biết của mình với mọi người cũng như tự giúp bản thân nắm vững hơn những gì đã đọc được.

Trong lập trình hướng đối tượng, 2 khái niệm object và data structure có sự đối lập nhau rõ ràng mặc dù nghe thì chúng có vẻ hơi hơi giống nhau. Trong bài viết này, mình sẽ tìm hiểu và phân tích sự đối lập này thông qua 1 ví dụ thực tế.

## Sự khác nhau giữa Object và Data Structure

Object đóng gói toàn bộ dữ liệu của nó bên trong 1 class và chỉ cung cấp ra bên ngoài một hoặc nhiều hàm để người dùng gọi. Như vậy, chỉ có các hàm bên trong 1 class mới có thể quản lý dữ liệu của class đó. Ngược lại, data structure để phơi bày dữ liệu của nó và không có bất kỳ hàm nào có ý nghĩa.

Ta có thể thấy trong đoạn văn trên, object và data structure hoàn toàn đối lập nhau. Một bên thì giấu dữ liệu và phơi bày hàm, còn một bên thì phơi bày dữ liệu và không có hàm (coi như giấu).

## Ví dụ

Mình hãy cùng xem xét 1 ví dụ như sau:
Dưới đây là 1 đoạn code viết theo hướng data structure. Mỗi shape chỉ đơn giản là 1 tập các thuộc tính riêng biệt của mỗi dạng hình học. Class Geometry có nhiệm vụ tính toán các giá trị chẳng hạn như diện tích dựa trên 3 shape này.
``` java
public class Square {
    public Point topLeft;
    public double side;
}

public class Rectangle {
    public Point topLeft;
    public double height;
    public double width;
}

public class Circle {
    public Point center;
    public double radius;
}

public class Geometry {
    public double area(Object shape) throws NoSuchShapeException {
        if (shape instanceof Square) {
            Square s = (Square)shape;
            return s.side * s.side;
        }
        else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle)shape;
            return r.height * r.width;
        }
        else if (shape instanceof Circle) {
            Circle c = (Circle)shape;
            return Math.PI * c.radius * c.radius;
        }
        throw new NoSuchShapeException();
    }
}
```
Còn dưới đây là 1 đoạn code được viết theo hướng OOP, mỗi shape bắt buộc phải thừa kế từ class Shape và tự hiện thực các hàm của Shape.
``` java
public class Square implements Shape {
    private Point topLeft;
    private double side;

    public double area() {
        return side * side;
    }
}

public class Rectangle implements Shape {
    private Point topLeft;
    private double height;
    private double width;

    public double area() {
        return height * width;
    }
}

public class Circle implements Shape {
    private Point center;
    private double radius;

    public double area() {
        return Math.PI * radius * radius;
    }
}
```

Giả sử như chúng ta muốn thêm 1 shape mới vào code (Triangle chẳng hạn), đối với đoạn code đầu tiên (code theo data structure) thì ta cần khai báo thêm 1 data structure mới tên là Triangle và thay đổi toàn bộ hàm có trong class Geometry (trong trường hợp này chỉ có 1 hàm area). Tuy nhiên, trong 1 trường hợp khác, khi ta chỉ muốn thêm 1 hàm mới để tính chu vi của shape, ta chỉ cần khai báo 1 hàm tên là perimeter trong class Geometry và mọi thay đổi chỉ gói gọn trong hàm này mà thôi.
Bây giờ mình sẽ dùng đoạn code thứ hai (sử dụng OOP) để áp dụng vào trường hợp trên. Việc thêm 1 shape mới trở nên dễ dàng hơn rất nhiều so với sử dụng data structure, chỉ cần tạo 1 class mới tên là Triangle và hiện thực các hàm cần thiết của class Shape mà không làm ảnh hưởng đến các Shape còn lại. Trong khi đó, nếu thêm hàm perimeter vào class Shape thì đòi hỏi ta phải thay đổi toàn bộ code của mọi class thừa kế từ Shape.

Như vậy có thể thấy rằng:
**Đối với lập trình sử dụng data structure, việc thêm 1 hàm mới trở nên dễ dàng mà không cần thay đổi code trong những data structure khác nhưng lại khó để thêm 1 data structure mới vì phải thay đổi toàn bộ hàm có sẵn. Ở chiều ngược lại, lập trình theo hướng đối tượng khiến cho việc thêm 1 class mới dễ dàng mà không cần thay đổi những class khác nhưng lại khó để thêm 1 hàm mới vì toàn bộ class khác phải thay đổi theo.**

