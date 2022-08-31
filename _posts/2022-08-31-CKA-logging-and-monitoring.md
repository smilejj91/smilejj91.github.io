---
title:  "CKA - logging and monitoring"

categories:
  - DevOps
tags:
  - cka
  - devops
  - k8s
---

### CKA - logging and monitoring

> Post에서 사용한 사진은 KODEKLOUD에 저작권이 있습니다.

#### 1. Commands

> kubectl top 명령어를 사용하기 위해서는 Metric Server가 필요함

> [metric-server](https://github.com/kubernetes-incubator/metrics-server.git)

```bash
$ kubectl top {object}
$ kubectl logs -f {pod name} {container name}
```

#### 2. Prometheus

> kubectl top을 통해서 CLI를 통해 cluster의 자원사용현황을 볼 수도 있지만, 좀 더 편리하게 자원을 모니터링하기 위해 prometheus와 grafana를 활용하는 게 좋음

> [prometheus-and-grafana-setting](https://github.com/smilejj91/k8s-cluster-setting/tree/main/app/prometheus)

> 이를 활용하면 다음과 같이 한눈에 보기 쉽게 자원 현황을 모니터링할 수 있으며, Grafana 대쉬보드는PromQL을 활용하여 자신의 입맛에 맞게 customizing 할 수 있음

> 보통은 사람들이 올려둔  dashboard template을 import해서 사용하거나, import 후 약간의 수정 가하여 사용하는  방향으로 많이 사용 (필자도 여기에 해당)

![K8S prometheus-and-grafana](/assets/img/k8s-prometheus-and-grafana.png)

