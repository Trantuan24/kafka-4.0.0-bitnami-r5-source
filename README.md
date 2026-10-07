# Source mapping for `bitnamilegacy/kafka:4.0.0-debian-12-r5`

Tài liệu này ghi lại mối quan hệ giữa hai bộ source trong thư mục `OpenSource` và image:

```text
bitnamilegacy/kafka:4.0.0-debian-12-r5
```

Repository tổng hợp này có cấu trúc:

```text
kafka-4.0.0-bitnami-r5-source/
├── README.md
├── apache-kafka-4.0.0/                  # source Kafka upstream
└── bitnami-kafka-4.0.0-debian-12-r5/   # Dockerfile và runtime scripts Bitnami
```

Hai thư mục con là snapshot source sạch tại các revision được ghi bên dưới; metadata `.git` của hai repository upstream không được lồng vào repository tổng hợp.

## Tổng quan mapping

```text
apache-kafka-4.0.0
  Apache Kafka tag 4.0.0
  commit 985bc99521dd22bbf620591b8db8613c54f596b2
                  │
                  │ mã nguồn logic Kafka upstream
                  ▼
  Kafka 4.0.0 binary/component: kafka-4.0.0-0
                  │
                  │ được Dockerfile Bitnami tải và cài vào /opt/bitnami/kafka
                  ▼
bitnami-kafka-4.0.0-debian-12-r5
  Bitnami containers commit bc79b90945d8f2acba584b5657c101072e5e0f1e
  + minideb:bookworm (Debian 12)
  + JRE 17.0.15-10-0
  + Kafka 4.0.0 component
  + Bitnami entrypoint, setup, run và thư viện cấu hình
                  │
                  ▼
bitnamilegacy/kafka:4.0.0-debian-12-r5
```

Hai thư mục không phải hai phiên bản Kafka khác nhau. `apache-kafka-4.0.0` chứa mã nguồn sản phẩm Kafka; `bitnami-kafka-4.0.0-debian-12-r5` chứa công thức đóng gói và các script vận hành để biến Kafka thành container Bitnami.

## 1. Source Apache Kafka

Thư mục:

```text
D:\12. VNPT-IT\OpenSource\apache-kafka-4.0.0
```

Thông tin đã ghim:

```text
Repository: https://github.com/apache/kafka.git
Tag:        4.0.0
Commit:     985bc99521dd22bbf620591b8db8613c54f596b2
License:    Apache License 2.0
```

Đây là source Java/Scala chính thức của Apache Kafka 4.0.0. Kafka 4.0 chỉ hỗ trợ KRaft; ZooKeeper mode đã bị loại bỏ. Broker, Kafka Connect và các tool của Kafka 4.0 yêu cầu Java 17.

## 2. Source đóng gói Bitnami

Source provenance ban đầu được lấy bằng sparse-checkout tại:

```text
D:\12. VNPT-IT\OpenSource\bitnami-kafka-4.0.0-debian-12-r5
```

Trong repository tổng hợp, source build Bitnami đã được đưa trực tiếp vào:

```text
D:\12. VNPT-IT\OpenSource\kafka-4.0.0-bitnami-r5-source\bitnami-kafka-4.0.0-debian-12-r5
```

Thông tin đã ghim:

```text
Repository: https://github.com/bitnami/containers.git
Release:    4.0.0-debian-12-r5
Commit:     bc79b90945d8f2acba584b5657c101072e5e0f1e
PR:         bitnami/containers#81232
License:    Apache License 2.0
```

Các thành phần quan trọng:

- `Dockerfile`: base image, JRE, Kafka component, OS packages, user và entrypoint.
- `prebuildfs/`: file được chép vào image trước khi cài Kafka component.
- `rootfs/`: entrypoint, setup, run script và thư viện cấu hình Bitnami.
- `docker-compose.yml`: ví dụ chạy container.
- `tags-info.yaml`: metadata của các tag image.

Dockerfile của revision này sử dụng:

```text
Base image:        docker.io/bitnami/minideb:bookworm
Operating system: Debian 12
Java component:   jre-17.0.15-10-0
Kafka component:  kafka-4.0.0-0
Kafka path:       /opt/bitnami/kafka
Runtime user:     1001
Entrypoint:       /opt/bitnami/scripts/kafka/entrypoint.sh
Command:          /opt/bitnami/scripts/kafka/run.sh
```

## Image đã đối chiếu

```text
Tag:            bitnamilegacy/kafka:4.0.0-debian-12-r5
Manifest digest: sha256:8a85b48e25ecd9a59db363b2ff89219e1c24cd5cf073bb66853c3b26c3bfa149
amd64 digest:    sha256:3b9e9b419fca250e8264afbac224ff5ce9282e41e73e26bbe92e7e6965c1f0d5
arm64 digest:    sha256:a6bd5d739a7fbe89099fb0bd706723b92db09df77add7ca542f8ca7969c15ea6
```

Để pull đúng manifest đã kiểm tra, không phụ thuộc việc tag có bị cập nhật hay không:

```powershell
docker pull bitnamilegacy/kafka@sha256:8a85b48e25ecd9a59db363b2ff89219e1c24cd5cf073bb66853c3b26c3bfa149
```

## Kiểm tra các clone provenance local ban đầu

```powershell
git -C "D:\12. VNPT-IT\OpenSource\apache-kafka-4.0.0" rev-parse HEAD
git -C "D:\12. VNPT-IT\OpenSource\apache-kafka-4.0.0" describe --tags --exact-match HEAD

git -C "D:\12. VNPT-IT\OpenSource\bitnami-kafka-4.0.0-debian-12-r5" rev-parse HEAD
```

Các lệnh trên kiểm tra hai clone provenance độc lập. Hai thư mục con trong repository tổng hợp không chứa `.git` riêng.

Kết quả mong đợi:

```text
985bc99521dd22bbf620591b8db8613c54f596b2
4.0.0
bc79b90945d8f2acba584b5657c101072e5e0f1e
```

## Build thử image từ source Bitnami

```powershell
cd "D:\12. VNPT-IT\OpenSource\kafka-4.0.0-bitnami-r5-source\bitnami-kafka-4.0.0-debian-12-r5"
docker build -t local/kafka:4.0.0-debian-12-r5 .
```

Build lại có thể tạo image tương đương về chức năng nhưng không đảm bảo trùng digest từng byte với image phát hành. Dockerfile tham chiếu base image bằng tag, tải component từ kho Bitnami và chạy `apt-get upgrade`; nội dung các nguồn này có thể đã thay đổi theo thời gian. Muốn triển khai đúng artifact đã phát hành thì dùng image digest ở trên. Hai bộ source được lưu để audit mã nguồn Kafka và logic đóng gói/container startup.

## Ghi chú về tên `bitnamilegacy`

Image ban đầu thuộc dòng Bitnami Kafka. Sau khi các image Debian thế hệ cũ được chuyển sang registry legacy, tag được cung cấp dưới tên `bitnamilegacy/kafka`. Việc đổi repository registry không biến nó thành một implementation Kafka khác: Kafka core vẫn là Apache Kafka 4.0.0, còn hành vi container đến từ lớp đóng gói Bitnami tại revision `r5`.
