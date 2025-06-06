# Пошаговый запуск тестов с Locust и Grafana

Ниже описаны шаги, которые помогут запустить мониторинг и эмуляцию нагрузки с помощью **Locust**, а затем просмотреть метрики в Grafana.

## 1. Сборка контейнера `locust_exporter`
```bash
git clone https://github.com/dduleba/locust_exporter.git
cd locust_exporter
docker build --tag locust_exporter .
```

## 2. Запуск мониторингового стека
Перед стартом необходимо указать IP адрес хоста, где будет работать Locust master. Проще всего взять адрес интерфейса `docker0`:
```bash
export LOCUST_HOST=`ip -4 addr show scope global dev docker0 | grep inet | awk '{print $2}' | cut -d / -f 1`
```
Затем запускаем сервисы Prometheus, Grafana и экспортёр Locust:
```bash
git clone https://github.com/dduleba/locust-dockprom.git
cd locust-dockprom
docker-compose up -d
```

## 3. Создание и запуск сценария нагрузки
В корне проекта имеется файл `locustfile.py` с минимальным примером сценария. Он выполняет GET запрос к `/`:
```python
from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 5)

    @task
    def index(self):
        self.client.get("/")
```
Запускаем мастер-процесс Locust (по умолчанию интерфейс доступен на `:8089`):
```bash
locust -f locustfile.py --master
```
При необходимости можно запустить воркеры:
```bash
locust -f locustfile.py --worker --master-host $LOCUST_HOST
```
Через веб‑интерфейс Locust или в headless‑режиме задайте количество пользователей и частоту запросов.

## 4. Просмотр метрик
Экспортёр публикует метрики на `:8080`. Prometheus собирает их автоматически. Grafana доступна по адресу `http://<host-ip>:3000` (логин/пароль `admin/admin`). Откройте дашборд **Locust** и наблюдайте графики во время выполнения теста.

Так можно получить реальные данные о производительности тестируемого сервиса и сохранить их для последующего анализа.
