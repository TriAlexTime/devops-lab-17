# Проверка развертывания LabServer в Minikube

## Состояние pod'ов

После применения Kubernetes-манифестов командой:

```bash
kubectl apply -f k8s/

в кластере были запущены три pod'а:

backend;
dashboard;
postgres.

Команда проверки:

kubectl get pods
показала, что все pod'ы находятся в состоянии Running.

Проверка сервисов

Команда:

kubectl get svc

показала наличие сервисов:

backend-service;
dashboard-service;
db-service.

Dashboard доступен через NodePort-сервис командой:

minikube service dashboard-service
Проверка livenessProbe

Для backend настроена проверка живости:

Liveness: tcp-socket :5288 delay=60s timeout=1s period=15s

Для dashboard настроена проверка живости:

Liveness: tcp-socket :4200 delay=120s timeout=1s period=15s

Таким образом, Kubernetes выполняет TCP-проверку портов контейнеров и может автоматически перезапустить контейнер при сбое.


Сохрани файл.

---

## Шаг 4. Не трогай пока ошибку с токеном

Окно авторизации в dashboard — это нормально.

Ошибка с отсутствующими таблицами:

```text
relation "StudentLabs" does not exist
relation "StudentLabSubmissions" does not exist