 ---
title: "Autoscaling CoreDNS"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 4.2 </b>"
---

#### Autoscaling CoreDNS

Trong môi trường Kubernetes, **CoreDNS** là dịch vụ DNS mặc định chạy trong các Pod với nhãn `k8s-app=kube-dns`. Trong bài thực hành này, chúng ta sẽ tự động điều chỉnh tỷ lệ CoreDNS dựa trên số lượng node có thể lên lịch và các lõi của cụm của chúng ta. **Cluster Proportional Autoscaler** sẽ thay đổi kích thước của các bản sao CoreDNS.

Hiện tại, chúng ta đang chạy một cụm 3 node:

```bash
$ kubectl get nodes
NAME                                            STATUS   ROLES    AGE   VERSION
ip-10-42-109-155.us-east-2.compute.internal   Ready    <none>   76m   vVAR::KUBERNETES_NODE_VERSION
ip-10-42-142-113.us-east-2.compute.internal   Ready    <none>   76m   vVAR::KUBERNETES_NODE_VERSION
ip-10-42-80-39.us-east-2.compute.internal     Ready    <none>   76m   vVAR::KUBERNETES_NODE_VERSION
```

Dựa trên các thông số tự động mở rộng được xác định trong `ConfigMap`, chúng ta thấy tỷ lệ tự động của cụm thay đổi CoreDNS thành 2 bản sao:

```bash
$ kubectl get po -n kube-system -l k8s-app=kube-dns
NAME                       READY   STATUS    RESTARTS   AGE
coredns-5db97b446d-5zwws   1/1     Running   0          66s
coredns-5db97b446d-n5mp4   1/1     Running   0          89m
```

Nếu chúng ta tăng kích thước của cụm EKS lên 5 node, **Cluster Proportional Autoscaler** sẽ tăng số bản sao của CoreDNS để phù hợp:

```bash hook=cpa-pod-scaleout timeout=300
$ aws eks update-nodegroup-config --cluster-name $EKS_CLUSTER_NAME \
  --nodegroup-name $EKS_DEFAULT_MNG_NAME --scaling-config desiredSize=$(($EKS_DEFAULT_MNG_DESIRED+2))

{
    "update": {
        "id": "52cd6682-ec47-3a9d-a77f-8d0b20cd84ae",
        "status": "InProgress",
        "type": "ConfigUpdate",
        "params": [
            {
                "type": "DesiredSize",
                "value": "5"
            }
        ],
        "createdAt": "2024-08-05T10:06:59.405000+00:00",
        "errors": []
    }
}
```

```bash hook=cpa-pod-scaleout timeout=300
$ aws eks wait nodegroup-active --cluster-name $EKS_CLUSTER_NAME \
  --nodegroup-name $EKS_DEFAULT_MNG_NAME
```

```bash hook=cpa-pod-scaleout timeout=300
$ kubectl wait --for=condition=Ready nodes --all --timeout=120s

node/ip-10-42-109-71.us-west-2.compute.internal condition met
node/ip-10-42-140-62.us-west-2.compute.internal condition met
node/ip-10-42-169-164.us-west-2.compute.internal condition met
node/ip-10-42-171-60.us-west-2.compute.internal condition met
node/ip-10-42-98-155.us-west-2.compute.internal condition met
```

Kubernetes hiện đang hiển thị 5 node ở trạng thái `Ready`:

```bash
$ kubectl get nodes
NAME                                          STATUS   ROLES    AGE   VERSION
ip-10-42-10-248.us-west-2.compute.internal    Ready    <none>   61s   vVAR::KUBERNETES_NODE_VERSION
ip-10-42-10-29.us-west-2.compute.internal     Ready    <none>   124m  vVAR::KUBERNETES_NODE_VERSION
ip-10-42-11-109.us-west-2.compute.internal    Ready    <none>   6m39s vVAR::KUBERNETES_NODE_VERSION
ip-10-42-11-152.us-west-2.compute.internal    Ready    <none>   61s   vVAR::KUBERNETES_NODE_VERSION
ip-10-42-12-139.us-west-2.compute.internal    Ready    <none>   6m20s vVAR::KUBERNETES_NODE_VERSION
```

Và chúng ta có thể thấy số lượng Pod CoreDNS đã tăng lên:

```bash
$ kubectl get po -n kube-system -l k8s-app=kube-dns
NAME                       READY   STATUS    RESTARTS   AGE
coredns-657694c6f4-klj6w   1/1     Running   0          14h
coredns-657694c6f4-tdzsd   1/1     Running   0          54s
coredns-657694c6f4-wmnnc   1/1     Running   0          14h
```

Bạn có thể xem nhật ký của CPA để xem cách nó phản ứng với sự thay đổi trong số lượng node trong cụm của chúng tôi:

```bash
$ kubectl logs deploy/dns-autoscaler -n other
{"includeUnschedulableNodes":true,"max":6,"min":2,"nodesPerReplica":2,"preventSinglePointFailure":true}
I0801 15:02:45.330307       1 k8sclient.go:272] Cluster status: SchedulableNodes[1], SchedulableCores[2]
I0801 15:02:45.330328       1 k8sclient.go:273] Replicas are not as expected : updating replicas from 2 to 3
```

Thông qua việc sử dụng **Cluster Proportional Autoscaler**, chúng ta có thể tự động điều chỉnh số lượng bản sao CoreDNS dựa trên kích thước và tài nguyên của cụm Kubernetes, giúp chúng ta duy trì tính sẵn sàng và hiệu suất của hệ thống DNS trong môi trường sản xuất.