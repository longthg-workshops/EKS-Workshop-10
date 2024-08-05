---
title: "Kubernetes Event-Driven Autoscaler (KEDA)"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 5. </b>"
---

Trước khi bắt đầu phần này, hãy chạy lệnh sau để chuẩn bị môi trường:

```bash
prepare-environment autoscaling/workloads/keda
```

Trong bài thực hành này, chúng ta sẽ tìm hiểu về việc sử dụng Kubernetes Event-Driven Autoscaler (KEDA) để mở rộng quy mô các pod trong tài nguyên deployment. Trong bài thực hành trước về Horizontal Pod Autoscaler (HPA), chúng ta đã thấy cách sử dụng tài nguyên HPA để mở rộng quy mô các pod theo chiều ngang trong deployment dựa trên mức sử dụng CPU trung bình. Nhưng đôi khi khối lượng công việc cần mở rộng quy mô dựa trên các sự kiện hoặc số liệu bên ngoài. KEDA cung cấp khả năng mở rộng quy mô khối lượng công việc của bạn dựa trên các sự kiện từ nhiều nguồn sự kiện khác nhau, chẳng hạn như độ dài hàng đợi trong Amazon SQS hoặc các số liệu khác trong CloudWatch. KEDA hỗ trợ hơn 60 trình mở rộng quy mô (scaler) cho các hệ thống số liệu, cơ sở dữ liệu, hệ thống nhắn tin, v.v.