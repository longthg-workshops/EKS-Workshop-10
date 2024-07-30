 ---
title: "Cluster Proportional Autoscaler"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 4. </b>"
---

#### Cluster Proportional Autoscaler

Trong bài lab này, chúng ta sẽ tìm hiểu về [Cluster Proportional Autoscaler](https://github.com/kubernetes-sigs/cluster-proportional-autoscaler) và cách tăng tỉ lệ của các ứng dụng theo tỉ lệ tính toán của cụm.

Cluster Proportional Autoscaler (CPA) là một horizontal pod autoscaler mà tăng số bản sao dựa trên số lượng node trong một cụm. Container tỷ lệ tỷ lệ tự động trong CPA theo dõi số lượng node có thể lên lịch và các core của một cụm và thay đổi kích thước của số bản sao. Chức năng này là mong muốn cho các ứng dụng cần phải tự động tăng tỉ lệ với kích thước của cụm như CoreDNS và các dịch vụ khác mà tăng tỉ lệ theo số lượng node/pod trong một cụm. CPA có các client API Golang chạy bên trong các pod kết nối với API Server và kiểm tra số lượng node và core trong cụm. Các tham số và điểm dữ liệu tỷ lệ được cung cấp qua một ConfigMap cho bộ tự động tỷ lệ và nó cập nhật bảng tham số của mình sau mỗi khoảng thời gian bỏ phiếu để có được dữ liệu mới nhất với các tham số tỷ lệ mong muốn. Khác với các bộ tự động tỷ lệ khác, CPA không phụ thuộc vào Metrics API và không yêu cầu Metrics Server.

![CPA](cpa.png)

Một số trường hợp sử dụng chính cho CPA bao gồm:

- Tăng tỷ lệ trên
- Mở rộng các dịch vụ nền tảng cốt lõi
- Cơ chế đơn giản và dễ dàng để tăng tỷ lệ công việc vì nó không yêu cầu metrics server hoặc prometheus adapter

#### Phương pháp Tăng tỷ lệ được sử dụng bởi Cluster Proportional Autoscaler

**Tuyến tính**

- Phương pháp tăng tỷ lệ này sẽ tăng ứng dụng theo tỷ lệ trực tiếp với số lượng node hoặc core có sẵn trong một cụm
- Một trong hai tham số `coresPerReplica` hoặc `nodesPerReplica` có thể được bỏ qua
- Khi `preventSinglePointFailure` được đặt thành `true`, bộ điều khiển đảm bảo ít nhất 2 bản sao nếu có nhiều hơn một node
- Khi `includeUnschedulableNodes` được đặt thành `true`, các bản sao sẽ được tăng tỷ lệ dựa trên tổng số node. Ngược lại, các bản sao chỉ sẽ tăng tỷ lệ dựa trên số lượng node có thể lên lịch (tức là, các node đã bị cordon và draining sẽ bị loại ra)
- Tất cả các tham số `min`, `max`, `preventSinglePointFailure`, `includeUnschedulableNodes` đều là tùy chọn. Nếu không được đặt, `min` sẽ được mặc định là 1, `preventSinglePointFailure` sẽ được mặc định là `false` và `includeUnschedulableNodes` sẽ được mặc định là `false`
- Cả `coresPerReplica` và `nodesPerReplica` là float

**ConfigMap cho Linear**

```
data:
  linear: |-
    {
      "coresPerReplica": 2,
      "nodesPerReplica": 1,
      "min": 1,
      "max": 100,
      "preventSinglePointFailure": true,
      "includeUnschedulableNodes": true
    }
```

**Công thức của Chế độ Kiểm soát Tuyến tính:**

```
replicas = max( ceil( cores * 1/coresPerReplica ) , ceil( nodes * 1/nodesPerReplica ) )
replicas = min(replicas, max)
replicas = max(replicas, min)
```

**Cầu thang**

- Phương pháp tăng tỷ lệ này sử dụng một hàm bước để xác định tỉ lệ node:bản sao và/hoặc core:bản sao
- Hàm cầu thang sử dụng điểm dữ liệu cho việc tự động tăng tỷ lệ của core và node từ ConfigMap. Bảng tra cứu mà cho ra số lượng bản sao cao hơn sẽ được sử dụng làm số lượng bản sao mục tiêu.
- Một trong hai tham số `coresPerReplica` hoặc `nodesPerReplica` có thể được bỏ qua
- Bản sao có thể được đặt thành 0 (không giống như trong chế độ tuyến tính)
- Tăng tỷ lệ lên 0 bản sao có thể được sử dụng để bật các tính năng tùy chọn khi một cụm phát triển

**ConfigMap cho Ladder**

```
data:
  ladder: |-
    {
      "coresToReplicas":
      [
        [ 1, 1 ],
        [ 64, 3 ],
        [ 512, 5 ],
        [ 1024, 7 ],
        [ 2048, 10 ],
        [ 4096, 15 ]
      ],
      "nodesToReplicas":
      [
        [ 1, 1 ],
        [ 2, 2 ]
      ]
    }
```

#### So sánh với Horizontal Pod Autoscaler

Horizontal Pod Autoscaler là một tài nguyên API cấp cao trong Kubernetes. HPA là một bộ tự động điều chỉnh

 phản hồi đóng mà giám sát sử dụng CPU/Bộ nhớ của các pod và tự động tăng số lượng bản sao. HPA dựa trên Metrics API và yêu cầu Metrics Server trong khi Cluster Proportional Autoscaler không sử dụng Metrics Server cũng như Metrics API. Cluster Proportional Autoscaler không tăng tỷ lệ với một nguồn tài nguyên Kubernetes mà thay vào đó sử dụng các cờ để xác định các công việc mục tiêu và một ConfigMap cho cấu hình tỷ lệ. CPA cung cấp một vòng lặp kiểm soát đơn giản mà theo dõi kích thước của cụm và tăng tỷ lệ của bộ điều khiển mục tiêu. Các đầu vào cho CPA là số lượng core và node có thể lên lịch trong cụm.