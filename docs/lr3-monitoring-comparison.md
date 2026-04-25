# Лабораторная работа №3  
## Сравнение систем мониторинга для Kubernetes-кластера

### 1. Цель работы

Целью лабораторной работы является выбор системы мониторинга для проекта LabServer, развернутого в Kubernetes-кластере.

Проект LabServer разворачивается как многокомпонентное приложение и включает:

- backend-сервис;
- frontend/dashboard-сервис;
- базу данных PostgreSQL.

В рамках практической части проект был запущен в локальном Kubernetes-кластере Minikube. После применения манифестов k8s/app.yaml и k8s/db.yaml были запущены pod'ы backend, dashboard и postgres.

Для backend и dashboard уже настроены Kubernetes livenessProbe:

- backend: TCP-проверка порта 5288;
- dashboard: TCP-проверка порта 4200.

Таким образом, Kubernetes может проверять живость контейнеров и автоматически перезапускать их при сбое.

### 2. Рассматриваемые системы мониторинга

Для сравнения были выбраны две системы мониторинга:

- Prometheus + Grafana;
- Zabbix.

Prometheus — система мониторинга, ориентированная на сбор time-series метрик. Она хорошо подходит для Kubernetes, так как умеет собирать метрики по pull-модели и интегрируется с Kubernetes service discovery.

Grafana используется совместно с Prometheus для визуализации метрик и построения дашбордов.

Zabbix — универсальная система мониторинга серверов, сетевого оборудования, сервисов и приложений. Она хорошо подходит для классической инфраструктуры, но для Kubernetes требует более сложной настройки.

### 3. Сравнение Prometheus и Zabbix

| Критерий | Prometheus + Grafana | Zabbix |
|---|---|---|
| Интеграция с Kubernetes | Очень высокая. Есть готовые Helm-чарты, kube-state-metrics, node-exporter, ServiceMonitor и PodMonitor | Возможна, но обычно требует дополнительных шаблонов, агентов или proxy |
| Модель сбора данных | Pull-модель: Prometheus сам опрашивает endpoints с метриками | Используются active/passive checks, агенты и внешние проверки |
| Развертывание в Kubernetes | Удобно разворачивается через Helm-чарт kube-prometheus-stack | Можно развернуть в Kubernetes, но настройка сложнее |
| Метрики pod'ов | Доступны через kube-state-metrics | Требуется дополнительная настройка источников данных |
| Визуализация | Grafana позволяет быстро создавать дашборды | Есть встроенные графики, также возможна интеграция с Grafana |
| Удобство для учебного проекта | Высокое: можно быстро получить метрики pod'ов, рестартов, CPU/RAM | Ниже: больше времени уходит на настройку |
| Типичный сценарий использования | Cloud-native и Kubernetes-мониторинг | Классический мониторинг серверов, сетей и сервисов |

### 4. Выбор системы мониторинга

Для проекта LabServer выбрана связка Prometheus + Grafana.

Причины выбора:

1. Prometheus является стандартным решением для мониторинга Kubernetes-кластеров.
2. Prometheus использует pull-модель, что удобно для динамической Kubernetes-среды.
3. Для Kubernetes существуют готовые exporters и компоненты, например kube-state-metrics и node-exporter.
4. Через Helm-чарт kube-prometheus-stack можно быстро развернуть Prometheus, Grafana, Alertmanager, kube-state-metrics и node-exporter.
5. Grafana позволяет наглядно отобразить состояние pod'ов, количество рестартов контейнеров и потребление ресурсов.

Zabbix также является мощной системой мониторинга, но для данной лабораторной работы Prometheus подходит лучше, так как он проще интегрируется с Kubernetes и быстрее позволяет получить нужные метрики.

### 5. Мониторинг живости контейнеров

В проекте LabServer мониторинг живости реализуется на двух уровнях.

#### 5.1. Уровень Kubernetes

В Kubernetes-манифестах используются livenessProbe.

LivenessProbe позволяет kubelet проверять, жив ли контейнер. Если проверка завершается ошибкой несколько раз подряд, Kubernetes считает контейнер неработоспособным и перезапускает его.

В текущем проекте используются TCP-проверки:

| Компонент | Тип проверки | Порт | Назначение |
|---|---|---:|---|
| backend | tcpSocket | 5288 | Проверка, что backend-приложение принимает соединения |
| dashboard | tcpSocket | 4200 | Проверка, что frontend/dashboard принимает соединения |
Пример обнаруженной настройки для backend:

Liveness: tcp-socket :5288 delay=60s timeout=1s period=15s #success=1 #failure=3

Пример обнаруженной настройки для dashboard:

Liveness: tcp-socket :4200 delay=120s timeout=1s period=15s #success=1 #failure=3

#### 5.2. Уровень Prometheus/Grafana

Prometheus будет использоваться для сбора метрик о состоянии Kubernetes-объектов.

Для этого используется kube-state-metrics. Этот компонент экспортирует метрики о состоянии pod'ов, deployment'ов, container'ов и других объектов Kubernetes.

Grafana будет использоваться для отображения этих метрик на дашборде.

### 6. Метрики для мониторинга

Для мониторинга состояния pod'ов и контейнеров будут использоваться следующие метрики:

| Назначение | Метрика | Описание |
|---|---|---|
| Проверка, что pod запущен | kube_pod_status_phase{phase="Running"} | Показывает pod'ы в состоянии Running |
| Проверка проблем с запуском | kube_pod_status_phase{phase="Pending"} | Показывает pod'ы, ожидающие запуска |
| Проверка аварийного состояния | kube_pod_status_phase{phase="Failed"} | Показывает pod'ы в состоянии Failed |
| Проверка неизвестного состояния | kube_pod_status_phase{phase="Unknown"} | Показывает pod'ы, состояние которых неизвестно |
| Количество рестартов контейнера | kube_pod_container_status_restarts_total | Показывает, сколько раз контейнер был перезапущен |

### 7. Планируемые PromQL-запросы

#### 7.1. Все pod'ы в состоянии Running

kube_pod_status_phase{phase="Running", namespace="default"}

#### 7.2. Только pod'ы проекта LabServer

kube_pod_status_phase{phase="Running", namespace="default", pod=~"backend.*|dashboard.*|postgres.*"}

#### 7.3. Количество рестартов контейнеров проекта

kube_pod_container_status_restarts_total{namespace="default", pod=~"backend.*|dashboard.*|postgres.*"}

#### 7.4. Количество Running pod'ов в namespace default

sum(kube_pod_status_phase{phase="Running", namespace="default"}) by (namespace)

#### 7.5. Pod'ы в нерабочем или проблемном состоянии

kube_pod_status_phase{phase=~"Failed|Unknown|Pending", namespace="default"}

### 8. Правила мониторинга

| Правило | Источник данных | Метрика / механизм | Условие | Действие |
|---|---|---|---|---|
| Pod должен быть запущен | kube-state-metrics | kube_pod_status_phase | phase="Running" равно 1 | Отображать pod как работающий |
| Pod не должен быть в Failed | kube-state-metrics | kube_pod_status_phase | phase="Failed" равно 1 | Отметить pod как аварийный |
| Pod не должен зависать в Pending | kube-state-metrics | kube_pod_status_phase | phase="Pending" длительное время | Проверить проблемы с образом, ресурсами или scheduling |
| Контейнер не должен часто перезапускаться | kube-state-metrics | kube_pod_container_status_restarts_total | значение увеличивается | Проверить причину рестартов |
| Backend должен отвечать на TCP-порту | Kubernetes livenessProbe | tcpSocket 5288 | проверка не проходит 3 раза подряд | Kubernetes перезапускает контейнер |
| Dashboard должен отвечать на TCP-порту | Kubernetes livenessProbe | tcpSocket 4200 | проверка не проходит 3 раза подряд | Kubernetes перезапускает контейнер |

### 9. Предполагаемый дашборд Grafana

В Grafana планируется создать дашборд со следующими панелями:

1. Статус pod'ов проекта LabServer:
   - backend;
   - dashboard;
   - postgres.

2. Количество рестартов контейнеров:
   - backend;
   - dashboard;
   - postgres.

3. Количество pod'ов в состоянии Running.

4. Наличие pod'ов в состояниях Pending, Failed или Unknown.

5. Потребление CPU и RAM pod'ами проекта, если соответствующие метрики будут доступны после установки kube-prometheus-stack.

### 10. Проверка отказоустойчивости

Для демонстрации работы Kubernetes self-healing можно выполнить тест:

kubectl delete pod -l app=backend

После удаления pod'а Kubernetes автоматически создаст новый pod на основе Deployment.

Ожидаемое поведение:
1. Старый backend pod будет удален.
2. Deployment создаст новый backend pod.
3. В Grafana будет видно изменение состояния pod'а.
4. Метрики рестартов и состояния pod'ов можно будет использовать для демонстрации реакции системы на сбой.

Аналогичный тест можно выполнить для dashboard:

kubectl delete pod -l app=dashboard

### 11. Примечание по текущему состоянию проекта

На текущем этапе backend, dashboard и PostgreSQL запускаются в Kubernetes и находятся в состоянии Running.

При просмотре логов backend обнаружено, что приложение обращается к таблицам StudentLabs и StudentLabSubmissions, которых пока нет в базе данных. Это означает, что база PostgreSQL поднята, но схема базы данных или seed-данные еще не инициализированы.

Эта проблема не блокирует выполнение части лабораторной работы, связанной с Kubernetes и мониторингом pod'ов, так как pod'ы запущены, сервисы созданы, livenessProbe работают, а dashboard открывается через NodePort-сервис.

### 12. Вывод

В результате сравнения систем мониторинга для проекта LabServer была выбрана связка Prometheus + Grafana.

Prometheus лучше подходит для Kubernetes-кластера, чем Zabbix, так как имеет удобную pull-модель сбора метрик, хорошо интегрируется с Kubernetes и может быть быстро развернут через Helm-чарт kube-prometheus-stack.

Grafana будет использоваться для визуализации состояния pod'ов, количества рестартов контейнеров и проверки реакции Kubernetes на сбои.

Таким образом, для лабораторной работы №4 будет использоваться следующая схема мониторинга:

Kubernetes cluster → kube-state-metrics → Prometheus → Grafana dashboard