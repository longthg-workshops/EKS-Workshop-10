---
title: "Generate load"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3.3 </b>"
---

#### Generate load

Để quan sát quá trình mở rộng với HPA (Horizontal Pod Autoscaler) phản ứng với chính sách đã cấu hình, cần tạo một khối lượng công việc trên ứng dụng của chúng ta. Chúng ta thực hiện điều đó bằng cách gọi trang chủ của công việc với [hey](https://github.com/rakyll/hey).

Lệnh dưới đây sẽ chạy công cụ tạo khối lượng công việc, bao gồm:

- 10 workers chạy đồng thời
- Gửi 5 truy vấn mỗi giây mỗi worker
- Chạy tối đa trong 60 phút

```bash hook=hpa-pod-scaleout hookTimeout=330
$ kubectl run load-generator \
  --image=williamyeh/hey:latest \
  --restart=Never -- -c 10 -q 5 -z 60m http://ui.ui.svc/home

pod/load-generator created
```

Bây giờ, khi các yêu cầu đến ứng dụng của chúng ta, chúng ta có thể theo dõi tài nguyên HPA để theo dõi tiến độ của nó:

```bash test=false
$ kubectl get hpa ui -n ui --watch
NAME   REFERENCE       TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
ui     Deployment/ui   69%/80%   1         4         1          117m
ui     Deployment/ui   99%/80%   1         4         1          117m
ui     Deployment/ui   89%/80%   1         4         2          117m
ui     Deployment/ui   89%/80%   1         4         2          117m
ui     Deployment/ui   84%/80%   1         4         3          118m
ui     Deployment/ui   84%/80%   1         4         3          118m
```

Khi bạn hài lòng với hành vi tự động mở rộng, bạn có thể kết thúc việc theo dõi với `Ctrl+C` và dừng công cụ tạo lượng tải như sau:

```bash timeout=180
$ kubectl delete pod load-generator
```

Khi công cụ tạo lượng công việc kết thúc, hãy chú ý rằng HPA sẽ từ từ đưa số bản sao về số tối thiểu dựa trên cấu hình của nó. Bạn có thể chạy lại lệnh `kubectl get hpa` để theo dõi điều này:
```bash
$ kubectl get hpa ui -n ui --watch

NAME   REFERENCE       TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
ui     Deployment/ui   cpu: 33%/80%   1         4         4          10m
ui     Deployment/ui   cpu: 26%/80%   1         4         4          10m
ui     Deployment/ui   cpu: 0%/80%    1         4         4          11m
ui     Deployment/ui   cpu: 1%/80%    1         4         4          11m
ui     Deployment/ui   cpu: 0%/80%    1         4         4          11m
ui     Deployment/ui   cpu: 0%/80%    1         4         4          11m
ui     Deployment/ui   cpu: 0%/80%    1         4         4          12m
ui     Deployment/ui   cpu: 0%/80%    1         4         4          12m
ui     Deployment/ui   cpu: 0%/80%    1         4         3          12m
ui     Deployment/ui   cpu: 0%/80%    1         4         3          12m
ui     Deployment/ui   cpu: 0%/80%    1         4         3          14m
ui     Deployment/ui   cpu: 0%/80%    1         4         2          14m
ui     Deployment/ui   cpu: 0%/80%    1         4         2          15m
ui     Deployment/ui   cpu: 0%/80%    1         4         1          16m
ui     Deployment/ui   cpu: 0%/80%    1         4         1          16m
```