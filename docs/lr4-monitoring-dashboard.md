# Лабораторная работа №4  
## Интеграция Prometheus и Grafana для мониторинга LabServer в Kubernetes

### 1. Цель работы

Целью практической части является развертывание системы мониторинга для проекта LabServer, работающего в Kubernetes-кластере Minikube.

Для мониторинга была выбрана связка:

- Prometheus;
- Grafana;
- kube-state-metrics;
- node-exporter;
- Alertmanager.

Развертывание выполнено через Helm-чарт kube-prometheus-stack.

### 2. Развертывание мониторинга

Для установки Prometheus и Grafana был создан отдельный namespace:

```bash
kubectl create namespace monitoring

Далее был добавлен Helm-репозиторий Prometheus Community:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

После этого был установлен стек мониторинга:

helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring --set grafana.adminPassword=admin12345

После установки были запущены следующие pod'ы:

monitoring-grafana;
monitoring-kube-prometheus-operator;
monitoring-kube-state-metrics;
monitoring-prometheus-node-exporter;
prometheus-monitoring-kube-prometheus-prometheus-0;
alertmanager-monitoring-kube-prometheus-alertmanager-0.

Проверка выполнялась командой:

kubectl get pods -n monitoring

Все pod'ы мониторинга перешли в состояние Running.

3. Доступ к Grafana

Для доступа к Grafana использовался port-forward:

kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80

После этого интерфейс Grafana был доступен по адресу:

http://localhost:3000

Использованные учетные данные:

login: admin
password: admin12345
4. Источник данных

В Grafana использовался источник данных Prometheus, который был автоматически создан при установке kube-prometheus-stack.

Сбор состояния Kubernetes-объектов выполняется через компонент kube-state-metrics.

Для проверки сбора метрик использовался PromQL-запрос:

kube_pod_status_phase{namespace="default"}

Он показал метрики для pod'ов проекта LabServer:

backend;
dashboard;
postgres.
5. Созданный dashboard

В Grafana был создан dashboard:

LabServer Kubernetes Monitoring

Dashboard содержит 4 панели.

5.1. Running pods LabServer

Панель показывает количество pod'ов проекта LabServer, находящихся в состоянии Running.

PromQL-запрос:

sum(kube_pod_status_phase{phase="Running", namespace="default", pod=~"backend.*|dashboard.*|postgres.*"})

На момент проверки значение равно 3, что означает, что все три pod'а проекта работают.

5.2. Pod Running Status

Панель показывает статус каждого pod'а отдельно.

PromQL-запрос:

kube_pod_status_phase{phase="Running", namespace="default", pod=~"backend.*|dashboard.*|postgres.*"}

Интерпретация:

1 — pod находится в состоянии Running;
0 — pod не находится в состоянии Running.

На момент проверки:

backend = 1;
dashboard = 1;
postgres = 1.
5.3. Container restarts

Панель показывает количество рестартов контейнеров.

PromQL-запрос:

sum by (pod) (kube_pod_container_status_restarts_total{namespace="default", pod=~"backend.*|dashboard.*|postgres.*"})

На момент проверки количество рестартов равно 0 для всех pod'ов.

5.4. Problem pods

Панель показывает количество pod'ов в проблемных состояниях:

Pending;
Failed;
Unknown.

PromQL-запрос:

sum(kube_pod_status_phase{phase=~"Pending|Failed|Unknown", namespace="default", pod=~"backend.*|dashboard.*|postgres.*"})

На момент проверки значение равно 0, то есть проблемных pod'ов нет.

6. Экспорт dashboard

Dashboard был экспортирован в JSON-файл:

grafana/labserver-k8s-monitoring-dashboard.json

Этот файл можно импортировать в Grafana на другом компьютере, чтобы получить такой же dashboard.

7. Результат

В результате была настроена система мониторинга Kubernetes-кластера для проекта LabServer.

Prometheus собирает метрики состояния pod'ов через kube-state-metrics, а Grafana отображает их на dashboard.

На момент проверки:

все 3 pod'а проекта находятся в состоянии Running;
проблемных pod'ов нет;
рестартов контейнеров нет;
dashboard успешно отображает состояние backend, dashboard и postgres.
8. Вывод
Связка Prometheus + Grafana успешно интегрирована с Kubernetes-кластером Minikube.

Использование Helm-чарта kube-prometheus-stack позволило быстро развернуть готовый стек мониторинга, включающий Prometheus, Grafana, kube-state-metrics, node-exporter и Alertmanager.

Созданный dashboard позволяет отслеживать живость pod'ов проекта LabServer, количество рестартов контейнеров и наличие проблемных состояний.


Сохрани файл через Ctrl + S.

---

# Шаг 3. Проверь файлы

В PowerShell выполни:

```powershell
dir docs
dir grafana

В docs должны быть:

lr3-monitoring-comparison.md
promql-queries.md
lr4-monitoring-dashboard.md

В grafana должен быть:

labserver-k8s-monitoring-dashboard.json