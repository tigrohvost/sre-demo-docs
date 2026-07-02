Алерт: `PodInfoHttpErrors`  
Severity: `WARN`  
Impact: Демонстрация эскалации на команду разработки

1. Подтверди проблему  
   Приложение в Kubernetes выдаёт ошибки, растёт время ответа

2. Проверь состояние кластера:  
  kubectl cluster-info
  # API server и базовые endpoints кластера

  kubectl get nodes -o wide
  # состояние нод, версии, IP, роли

  kubectl get pods -A -o wide
  # все Pod'ы во всех namespace, рестарты и размещение по нодам

  kubectl get deploy,sts,ds -A
  # готовность основных workloads: Deployment, StatefulSet, DaemonSet

  kubectl get events -A --sort-by=.lastTimestamp
  # последние события: ошибки scheduling, pull image, restarts, probes
   
