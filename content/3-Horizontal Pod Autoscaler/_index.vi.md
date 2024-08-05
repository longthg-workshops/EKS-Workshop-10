---
title: "Horizontal Pod Autoscaler"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3. </b>"
---

#### Horizontal Pod Autoscaler

Trong bài lab này, chúng ta sẽ tìm hiểu về **Horizontal Pod Autoscaler** (HPA) để điều chỉnh tỷ lệ pod trong một deployment hoặc replica set. Nó được thực hiện như một tài nguyên API của **Kubernetes** và một controller. Tài nguyên xác định hành vi của controller. Bộ quản lý Controller thực hiện truy vấn sử dụng tài nguyên đối với các chỉ số được xác định trong mỗi định nghĩa HorizontalPodAutoscaler. Controller định kỳ điều chỉnh số lượng bản sao trong một replication controller hoặc deployment đến mục tiêu được chỉ định bởi người dùng bằng cách quan sát các chỉ số như sử dụng CPU trung bình, sử dụng bộ nhớ trung bình hoặc bất kỳ chỉ số tùy chỉnh nào khác. Nó lấy các chỉ số từ API tài nguyên (cho các chỉ số tài nguyên trên mỗi pod), hoặc API chỉ số tùy chỉnh (cho tất cả các chỉ số khác).

**Bước chuẩn bị môi trường cho phần này:**

```bash
$ prepare-environment autoscaling/workloads/hpa
```

Điều này sẽ thực hiện các thay đổi sau đây trong môi trường lab của bạn:

- Cài đặt **Kubernetes Metrics Server** trong cluster **Amazon EKS**

**Kubernetes Metrics Server** là một bộ tổng hợp có khả năng mở rộng và hiệu quả về dữ liệu sử dụng tài nguyên trong cluster của bạn. Nó cung cấp các chỉ số container cần thiết cho **Horizontal Pod Autoscaler**. Metrics server không được triển khai mặc định trong các cluster **Amazon EKS**.

![EKS](/EKS-Workshop-10/images/0007/0001.png?featherlight=false&width=60pc)