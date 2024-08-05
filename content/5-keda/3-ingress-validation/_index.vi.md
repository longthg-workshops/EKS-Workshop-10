---
title: "Xác mình Ingress"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 5.3 </b>"
---

### Xác mình Ingress

Như một phần của các điều kiện tiên quyết của bài thực hành, một tài nguyên Ingress đã được tạo và AWS Load Balancer Controller đã tạo một ALB tương ứng dựa trên cấu hình Ingress. Sẽ mất vài phút để ALB cung cấp và đăng ký các mục tiêu của nó. Hãy xác thực tài nguyên Ingress và ALB trước khi tiếp tục.

```bash
$ kubectl get ingress ui -n ui

NAME   CLASS   HOSTS   ADDRESS                                                      PORTS   AGE
ui     alb     *       k8s-ui-ui-5ddc3ba496-107943159.us-west-2.elb.amazonaws.com   80      3m51s
```

Chờ đến khi thành phần cân bằng tải được cung cấp xong. Sau đó chạy lệnh sau:

```bash
$ wait-for-lb $(kubectl get ingress -n ui ui -o jsonpath="{.status.loadBalancer.ingress[*].hostname}{'\n'}")

Waiting for k8s-ui-ui-5ddc3ba496-107943159.us-west-2.elb.amazonaws.com...
You can now access http://k8s-ui-ui-5ddc3ba496-107943159.us-west-2.elb.amazonaws.com
```