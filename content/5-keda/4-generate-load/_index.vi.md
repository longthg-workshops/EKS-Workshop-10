---
title: "Tạo khối lượng công việc"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 5.4 </b>"
---

### Tạo khối lượng công việc

Để quan sát KEDA mở rộng delployment để đáp ứng `ScaledObject` đã được cấu hình, chúng ta cần tạo một khối lượng công việc trên ứng dụng. Chúng ta sẽ làm điều đó bằng cách gọi trang chủ của khối lượng công việc với lệnh [hey](https://github.com/rakyll/hey).

Lệnh bên dưới sẽ chạy trình tạo khối lượng công việc với:

1. 3 worker chạy đồng thời
2. Gửi 5 truy vấn mỗi giây
3. Chạy trong tối đa 10 phút

```bash
$ export ALB_HOSTNAME=$(kubectl get ingress ui -n ui -o yaml | yq .status.loadBalancer.ingress[0].hostname)

$ kubectl run load-generator \
  --image=williamyeh/hey:latest \
  --restart=Never -- -c 3 -q 5 -z 10m http://$ALB_HOSTNAME/home
```

Dựa vào ScaledObject, KEDA tạo ra một tài nguyên HPA và cung cấp các số liệu cần thiết để cho phép HPA mở rộng khối lượng công việc. Bây giờ chúng ta có các yêu cầu đến ứng dụng của mình, chúng ta có thể theo dõi tài nguyên HPA để theo dõi tiến trình của nó:

```bash
$ kubectl get hpa keda-hpa-ui-hpa -n ui --watch

NAME              REFERENCE       TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
keda-hpa-ui-hpa   Deployment/ui   7/100 (avg)   1         10        1          7m58s
keda-hpa-ui-hpa   Deployment/ui   778/100 (avg)   1         10        1          8m33s
keda-hpa-ui-hpa   Deployment/ui   194500m/100 (avg)   1         10        4          8m48s
keda-hpa-ui-hpa   Deployment/ui   97250m/100 (avg)    1         10        8          9m3s
keda-hpa-ui-hpa   Deployment/ui   625m/100 (avg)      1         10        8          9m18s
keda-hpa-ui-hpa   Deployment/ui   91500m/100 (avg)    1         10        8          9m33s
keda-hpa-ui-hpa   Deployment/ui   92125m/100 (avg)    1         10        8          9m48s
keda-hpa-ui-hpa   Deployment/ui   750m/100 (avg)      1         10        8          10m
keda-hpa-ui-hpa   Deployment/ui   102625m/100 (avg)   1         10        8          10m
keda-hpa-ui-hpa   Deployment/ui   113625m/100 (avg)   1         10        8          11m
keda-hpa-ui-hpa   Deployment/ui   90900m/100 (avg)    1         10        10         11m
keda-hpa-ui-hpa   Deployment/ui   91500m/100 (avg)    1         10        10         12m
```

Khi bạn đã hài lòng với hành vi tự động điều chỉnh, bạn có thể kết thúc việc theo dõi bằng Ctrl+C và dừng trình tạo công việc như sau:

```bash
$ kubectl delete pod load-generator
```

Khi công cụ tạo lượng công việc kết thúc, hãy chú ý rằng HPA sẽ từ từ đưa số bản sao về số tối thiểu dựa trên cấu hình của nó.