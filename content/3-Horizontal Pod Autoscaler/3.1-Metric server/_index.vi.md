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

NAME                              READY   STATUS    RESTARTS   AGE
metrics-server-7577444cf8-5bph6   1/1     Running   0          11m
```

Để có cái nhìn về các số liệu mà **HPA** sẽ sử dụng để điều chỉnh hành vi tự động mở rộng của nó, bạn có thể sử dụng lệnh `kubectl top`. Ví dụ, lệnh này sẽ hiển thị việc sử dụng tài nguyên của các node trong cụm của chúng ta:

```bash
$ kubectl top node

NAME                                          CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
ip-10-42-109-71.us-west-2.compute.internal    22m          1%     896Mi           12%       
ip-10-42-140-62.us-west-2.compute.internal    30m          1%     1367Mi          19%       
ip-10-42-169-164.us-west-2.compute.internal   179m         9%     1375Mi          19% 
```

Bạn cũng có thể lấy thông tin sử dụng tài nguyên của các pod, ví dụ:

```bash
$ kubectl top pod -l app.kubernetes.io/created-by=eks-workshop -A

NAMESPACE   NAME                              CPU(cores)   MEMORY(bytes)   
assets      assets-784b5f5656-x8c2v           1m           2Mi             
carts       carts-6887ddd44c-ktv2v            4m           210Mi           
carts       carts-dynamodb-69fc586887-wl2sn   1m           139Mi           
catalog     catalog-5578f9649b-r5cfs          1m           17Mi            
catalog     catalog-mysql-0                   2m           390Mi           
checkout    checkout-84c6769ddd-jq9mc         9m           76Mi            
checkout    checkout-redis-76bc7cb6f9-62q9c   1m           2Mi             
orders      orders-6d74499d87-vwb2q           4m           243Mi           
orders      orders-mysql-6fbd688d4b-vvrxs     2m           392Mi           
ui          ui-5f4d85f85f-wtd79               1m           177Mi  
```

Khi chúng ta quan sát **HPA** mở rộng các pod, bạn có thể tiếp tục sử dụng các truy vấn này để hiểu rõ hơn về những gì đang diễn ra.