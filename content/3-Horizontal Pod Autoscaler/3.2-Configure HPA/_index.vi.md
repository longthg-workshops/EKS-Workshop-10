---
title: "Cấu hình HPA"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 3.2 </b>"
---

#### Cấu hình HPA

Hiện tại không có tài nguyên nào trong cụm EKS của chúng ta cho phép tự động mở rộng pod theo chiều ngang, bạn có thể kiểm tra bằng lệnh sau:

```bash expectError=true
$ kubectl get hpa -A

No resources found
```

Trong trường hợp này, chúng ta sẽ sử dụng dịch vụ `ui` và mở rộng dựa trên việc sử dụng CPU. Điều đầu tiên cần làm là cập nhật đặc tả Pod `ui` để chỉ định các giá trị `request` và `limit` của CPU.

```kustomization
modules/autoscaling/workloads/hpa/deployment.yaml
Deployment/ui
```

Tiếp theo, chúng ta cần tạo một tài nguyên `HorizontalPodAutoscaler` để xác định các tham số được HPA sử dụng để quyết định cách mở rộng khối lượng công việc của chúng ta.

```file
manifests/modules/autoscaling/workloads/hpa/hpa.yaml
```

Hãy áp dụng cấu hình này:

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/autoscaling/workloads/hpa

namespace/assets unchanged
namespace/carts unchanged
namespace/catalog unchanged
namespace/checkout unchanged
namespace/orders unchanged
namespace/other unchanged
namespace/rabbitmq unchanged
namespace/ui unchanged
serviceaccount/assets unchanged
serviceaccount/carts unchanged
serviceaccount/catalog unchanged
serviceaccount/checkout unchanged
serviceaccount/orders unchanged
serviceaccount/rabbitmq unchanged
serviceaccount/ui unchanged
role.rbac.authorization.k8s.io/rabbitmq-endpoint-reader unchanged
rolebinding.rbac.authorization.k8s.io/rabbitmq-endpoint-reader unchanged
configmap/assets unchanged
configmap/carts unchanged
configmap/catalog unchanged
configmap/checkout unchanged
configmap/orders unchanged
configmap/dummy unchanged
configmap/ui unchanged
secret/catalog-db unchanged
secret/orders-db unchanged
secret/rabbitmq unchanged
secret/rabbitmq-config unchanged
service/assets unchanged
service/carts unchanged
service/carts-dynamodb unchanged
service/catalog unchanged
service/catalog-mysql unchanged
service/checkout unchanged
service/checkout-redis unchanged
service/orders unchanged
service/orders-mysql unchanged
service/rabbitmq configured
service/rabbitmq-headless unchanged
service/ui unchanged
deployment.apps/assets unchanged
deployment.apps/carts unchanged
deployment.apps/carts-dynamodb unchanged
deployment.apps/catalog unchanged
deployment.apps/checkout unchanged
deployment.apps/checkout-redis unchanged
deployment.apps/orders unchanged
deployment.apps/orders-mysql unchanged
deployment.apps/ui configured
statefulset.apps/catalog-mysql unchanged
statefulset.apps/rabbitmq configured
horizontalpodautoscaler.autoscaling/ui created
```
