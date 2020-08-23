---
title: Policy Gradient cho bài toán Reinforcement Learning
date: 2020-08-12 12:50:25
tags:
- Machine Learning
- Deep Learning
- AI
- Reinforcement Learning
- Policy Gradient
---
Policy Gradient là 1 giải thuật để giải quyết các bài toán trong Reinforcement Learning.
Ý tưởng của giải thuật này rất đơn giản, đó là:
> "Quan sát và hành động"

Về cơ bản, agent của chúng ta tiếp thu những quan sát về trạng thái hiện tại của nó từ environment và sinh ra hành động cụ thể. Ta có 1 policy để điều khiển hành vi của agent dựa trên những quan sát đó. Nó khác với các giải thuật value-based như Q-Learning, Deep Q-Learning ở chỗ là những giải thuật này chọn ra hành động có phần thưởng mong đợi trong tương lai (expected future reward) cao nhất mà mô hình dự đoán được và cố gắng tối ưu hóa mô hình dự đoán này sao cho đúng với giá trị thực tế. Trong khi đó, Policy Gradient nói riêng và các giải thuật policy-based nói chung cho mô hình tính toán ra xác suất thực hiện của từng hành động tại một trạng thái nhất định và cố gắng tối ưu hóa tổng phần thưởng cho agent bằng cách thay đổi tham số của mô hình policy.

Ví dụ:

Trong trò chơi đào vàng, agent (người chơi) có 6 lựa chọn hành động (đi trái, đi phải, đi lên, đi xuống, đào vàng, nghỉ ngơi). Policy Gradient Agent sẽ output xác suất tương ứng với từng hành động tại trạng thái hiện tại. Nếu agent đang đứng trên mỏ vàng, xác suất cho hành động đào vàng sẽ cao hơn (giả sử như 80%, các hành động còn lại có xác suất là 4%). Điều này có nghĩa là agent sẽ có 80% khả năng sẽ thực hiện hành động đào vàng khi ở trạng thái này và có 4% thực hiện các hành động khác. Còn trong Deep Q-Learning, nếu không tính đến giải thuật epsilon-greedy thì agent luôn chọn 1 hành động có phần thưởng mong đợi trong tương lai cao nhất.

{% asset_img Game-Ver3-01.png This is an example image %}

