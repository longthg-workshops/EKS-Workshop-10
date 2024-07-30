 ---
title: "Cấu hình HPA"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 3.2 </b>"
---

#### Cấu hình HPA

Hiện tại không có tài nguyên nào trong cluster của chúng ta cho phép tự động mở rộng pod theo chiều ngang, bạn có thể kiểm tra bằng lệnh sau:

```bash expectError=true
$ kubectl get hpa -A
No resources found
```

Trong trường hợp này, chúng ta sẽ sử dụng dịch vụ `ui` và mở rộng dựa trên việc sử dụng CPU. Điều đầu tiên chúng ta sẽ làm là cập nhật đặc tả Pod `ui` để chỉ định các giá trị `request` và `limit` của CPU.

```kustomization
modules/autoscaling/workloads/hpa/deployment.yaml
Deployment/ui
```

Tiếp theo, chúng ta cần tạo một tài nguyên `HorizontalPodAutoscaler` để xác định các tham số mà HPA sẽ sử dụng để quyết định cách mở rộng tải của chúng ta.

```file
manifests/modules/autoscaling/workloads/hpa/hpa.yaml
```

Hãy áp dụng cấu hình này:

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/autoscaling/workloads/hpa
```
