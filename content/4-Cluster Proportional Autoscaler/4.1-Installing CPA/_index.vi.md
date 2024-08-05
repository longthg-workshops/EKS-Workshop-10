 ---
title: "Cài đặt CPA"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 4.1 </b>"
---

#### Cài đặt CPA

Trong bài thực hành này, chúng ta sẽ cài đặt CPA (Cluster Proportional Autoscaler) bằng cách sử dụng các tài liệu Kustomize, phần chính của nó là tài nguyên `Deployment` dưới đây:

```file
manifests/modules/autoscaling/workloads/cpa/deployment.yaml
```

Chúng ta sẽ sử dụng Helm chart để cài đặt CPA. Trước tiên cần thêm kho `cluster-proportional-autoscaler`:

```bash
$ helm repo add cluster-proportional-autoscaler https://kubernetes-sigs.github.io/cluster-proportional-autoscaler

"cluster-proportional-autoscaler" has been added to your repositories
```

Sau đó nâng cấp hoặc cài đặt với lệnh sau:

```bash
$ helm upgrade --install cluster-proportional-autoscaler cluster-proportional-autoscaler/cluster-proportional-autoscaler \
  --namespace kube-system \
  --version "${CPA_CHART_VERSION}" \
  --set "image.tag=v${CPA_VERSION}" \
  --values ~/environment/eks-workshop/modules/autoscaling/workloads/cpa/values.yaml \
  --wait

NAME: cluster-proportional-autoscaler
LAST DEPLOYED: [...]
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

Điều này sẽ tạo ra một `Deployment` trong namespace `kube-system`. Chúng ta có thể kiểm tra với lệnh sau:

```bash
$ kubectl get deployment dns-autoscaler -n other
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
dns-autoscaler   1/1     1            1           10s
```

Sau khi CPA khởi động, nó sẽ tự động tạo ra một `ConfigMap` có thể được sửa đổi để điều chỉnh cấu hình của nó:

```bash
$ kubectl describe configmap dns-autoscaler -n kube-system
Name:         dns-autoscaler
Namespace:    other
Labels:       <none>
Annotations:  <none>

Data
====
linear:
----
{'coresPerReplica':2,'includeUnschedulableNodes':true,'nodesPerReplica':1,'preventSinglePointFailure':true,'min':1,'max':4}
Events:  <none>
```

Qua bài viết này, bạn đã biết cách cài đặt và cấu hình CPA trên môi trường AWS EKS sử dụng Kustomize.