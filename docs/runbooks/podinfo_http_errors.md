## Runbook: PodInfoHttpErrors

при алерте `PodInfoHttpErrors`.
В этом проекте PodInfo работает как `deployment/podinfo` в namespace `demo-alert`,
ожидаемое состояние - 4 готовых pod-а. 

### 1. Подтвердить алерт

```bash
kubectl --namespace prometheus port-forward service/kube-prometheus-stack-prometheus 9090:9090
kubectl --namespace prometheus port-forward service/kube-prometheus-stack-alertmanager 9093:9093

curl -sS http://127.0.0.1:9090/api/v1/alerts \
  | jq -r '.data.alerts[]? | select(.labels.alertname=="PodInfoHttpErrors") | [.state, .labels.severity, .labels.namespace, .labels.service, .annotations.summary] | @tsv'
```

Комментарий: `firing` значит, что проблема сейчас активна.


### 2. Проверить Deployment и pod-ы

```bash
kubectl --namespace demo-alert get deployment podinfo
kubectl --namespace demo-alert get pods -l app=podinfo -o wide
kubectl --namespace demo-alert describe deployment podinfo
```

Комментарий: нормальное состояние - `READY 4/4`, pod-ы `Running` и `Ready`.
Если видны `CrashLoopBackOff`, частые restarts или pod-ы не Ready, сначала
смотри события и логи.

### 3. Посмотреть логи и Kubernetes-события

```bash
kubectl --namespace demo-alert logs deployment/podinfo --since=15m --tail=100
kubectl --namespace demo-alert get events --sort-by=.lastTimestamp | tail -30
kubectl --namespace demo-alert describe pods -l app=podinfo
```

Комментарий: 5xx, ошибки readiness/liveness probe, `OOMKilled`, проблемы с
pull image, рестарты контейнера или сообщения про node.


### 4. Gерезапустить PodInfo

```bash
kubectl --namespace demo-alert rollout restart deployment/podinfo
kubectl --namespace demo-alert rollout status deployment/podinfo --timeout=2m
kubectl --namespace demo-alert get pods -l app=podinfo -o wide
```

Комментарий: у Deployment настроен rolling update, поэтому Kubernetes будет
заменять pod-ы постепенно.

### 5. проверить восстановление

```bash
kubectl --namespace demo-alert get deployment podinfo

curl -sS http://127.0.0.1:9090/api/v1/alerts \
  | jq -r '.data.alerts[]? | select(.labels.alertname=="PodInfoHttpErrors") | [.state, .labels.namespace, .labels.service] | @tsv'

curl -sS http://127.0.0.1:9093/api/v2/alerts \
  | jq -r '.[] | select(.labels.alertname=="PodInfoHttpErrors") | {labels:.labels,status:.status,receivers:.receivers}'
```

Комментарий: если алерт ушел в resolved - зафиксируй время восстановления и
причину. 
