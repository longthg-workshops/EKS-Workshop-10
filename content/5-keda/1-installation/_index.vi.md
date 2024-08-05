---
title: "Cài đặt KEDA"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 5.1 </b>"
---

### Cài đặt KEDA

Trước tiên, ta cài đặt KEDA bằng Helm. Một điều kiện tiên quyết đã được tạo trong giai đoạn chuẩn bị phòng thí nghiệm: IAM Role đã được tạo với quyền truy cập dữ liệu số liệu trong CloudWatch.

```bash
helm repo add kedacore https://kedacore.github.io/charts
```

```bash
$ helm upgrade --install keda kedacore/keda \
  --version "${KEDA_CHART_VERSION}" \
  --namespace keda \
  --create-namespace \
  --set "podIdentity.aws.irsa.enabled=true" \
  --set "podIdentity.aws.irsa.roleArn=${KEDA_ROLE_ARN}" \
  --wait

Release "keda" does not exist. Installing it now.
NAME: keda
LAST DEPLOYED: [...]
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
[...]
```

Sau khi cài đặt Helm, KEDA sẽ chạy dưới dạng một số bản Deployment trong không gian tên keda:
```bash
$ kubectl get deployment -n keda

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
keda-admission-webhooks           1/1     1            1           105s
keda-operator                     1/1     1            1           105s
keda-operator-metrics-apiserver   1/1     1            1           105s
```

Mỗi deployment có một vai trò khác nhau:

1. Agent (keda-operator) - kiểm soát việc mở rộng khối lượng công việc
2. Metrics (keda-operator-metrics-server) - hoạt động như một máy chủ số liệu Kubernetes, cung cấp quyền truy cập vào số liệu bên ngoài
3. Admission Webhooks (keda-admission-webhooks) - xác thực cấu hình tài nguyên để ngăn ngừa cấu hình sai (ví dụ: nhiều ScaledObject nhắm mục tiêu vào cùng một khối lượng công việc)