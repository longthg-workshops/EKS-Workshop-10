 ---
title: "Installing CPA"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 4.1 </b>"
---

#### Installing CPA

Trong bài thực hành này, chúng ta sẽ cài đặt CPA (Cluster Proportional Autoscaler) bằng cách sử dụng các tài liệu Kustomize, phần chính của nó là tài nguyên `Deployment` dưới đây:

```file
manifests/modules/autoscaling/workloads/cpa/deployment.yaml
```

Hãy áp dụng điều này vào cluster của chúng ta:

```bash hook=cpa-install timeout=180
$ kubectl apply -k ~/environment/eks-workshop/modules/autoscaling/workloads/cpa
```

Điều này sẽ tạo ra một `Deployment` trong namespace `kube-system` mà chúng ta có thể kiểm tra:

```bash
$ kubectl get deployment dns-autoscaler -n other
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
dns-autoscaler   1/1     1            1           10s
```

Sau khi CPA khởi động, nó sẽ tự động tạo ra một `ConfigMap` mà chúng ta có thể sửa đổi để điều chỉnh cấu hình của nó:

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