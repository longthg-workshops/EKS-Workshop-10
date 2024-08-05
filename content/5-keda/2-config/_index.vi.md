---
title: "Cấu hình KEDA"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 5.2 </b>"
---

### Cấu hình KEDA

Khi được cài đặt, KEDA tạo ra một số tài nguyên tùy chỉnh. Một trong những tài nguyên đó, ScaledObject, cho phép ánh xạ nguồn sự kiện bên ngoài tới Deployment hoặc StatefulSet để mở rộng quy mô. Trong bài thực hành này, chúng ta sẽ tạo một ScaledObject nhắm mục tiêu đến Deployment `ui ` và mở rộng quy mô khối lượng công việc này dựa trên số liệu RequestCountPerTarget trong CloudWatch.

Đường dẫn tệp cấu hình:

```file
~/environment/eks-workshop/modules/autoscaling/workloads/keda/scaledobject/scaledobject.yaml
```

Trước tiên, chúng ta cần thu thập một số thông tin về Application Load Balancer (ALB) và Target Group được tạo ra như một phần của điều kiện tiên quyết trong bài thực hành.

```bash
$ export ALB_ARN=$(aws elbv2 describe-load-balancers --query 'LoadBalancers[?contains(LoadBalancerName, `k8s-ui-ui`) == `true`]' | jq -r .[0].LoadBalancerArn)

$ export ALB_ID=$(aws elbv2 describe-load-balancers --query 'LoadBalancers[?contains(LoadBalancerName, `k8s-ui-ui`) == `true`]' | jq -r .[0].LoadBalancerArn | awk -F "loadbalancer/" '{print $2}')

$ export TARGETGROUP_ID=$(aws elbv2 describe-target-groups --load-balancer-arn $ALB_ARN | jq -r '.TargetGroups[0].TargetGroupArn' | awk -F ":" '{print $6}')
```

Giờ chúng ta có thể sử dụng các giá trị đó để cập nhật cấu hình của ScaledObject và tạo tài nguyên trong cụm:

```bash
$ kubectl kustomize ~/environment/eks-workshop/modules/autoscaling/workloads/keda/scaledobject \
  | envsubst | kubectl apply -f-
```