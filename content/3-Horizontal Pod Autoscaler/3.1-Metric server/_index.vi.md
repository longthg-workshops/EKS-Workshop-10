 ---
title: "Metric server"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 3.1 </b>"
---

#### Metric server

Trong môi trường Kubernetes, **Kubernetes Metrics Server** đóng vai trò là một bộ tập hợp dữ liệu sử dụng tài nguyên trong cụm của bạn và nó không được triển khai mặc định trong các cụm Amazon EKS. Để biết thêm thông tin, bạn có thể xem [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server) trên GitHub. **Metrics Server** thường được sử dụng bởi các tiện ích bổ sung của Kubernetes khác, như Horizontal Pod Autoscaler hoặc Kubernetes Dashboard. Để biết thêm thông tin, bạn có thể xem [Resource metrics pipeline](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/) trong tài liệu Kubernetes. Trong bài thực hành này, chúng ta sẽ triển khai **Kubernetes Metrics Server** trên cụm Amazon EKS của chúng ta.

**Metric Server** đã được thiết lập sẵn trong cụm của chúng ta cho workshop này:

```bash
$ kubectl -n kube-system get pod -l app.kubernetes.io/name=metrics-server
```

Để có cái nhìn về các số liệu mà **HPA** sẽ sử dụng để điều chỉnh hành vi tự tự động mở rộng của nó, bạn có thể sử dụng lệnh `kubectl top`. Ví dụ, lệnh này sẽ hiển thị việc sử dụng tài nguyên của các node trong cụm của chúng ta:

```bash
$ kubectl top node
```

Bạn cũng có thể lấy được việc sử dụng tài nguyên của các pod, ví dụ:

```bash
$ kubectl top pod -l app.kubernetes.io/created-by=eks-workshop -A
```

Khi chúng ta quan sát **HPA** mở rộng các pod, bạn có thể tiếp tục sử dụng các truy vấn này để hiểu rõ hơn về những gì đang diễn ra.