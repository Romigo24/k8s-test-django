## Развертывание в Yandex Cloud (Kubernetes кластер yc-sirius-dev)

### Инструкции для деплоя Django-сайта в dev-окружении Kubernetes (`namespace: edu-roman-kolyada`). 

Сайт доступен по адресу:  [![Demo Version](https://img.shields.io/badge/Cайт-%E2%86%92_edu--roman--kolyada.yc--sirius--dev.pelid.team-blue)](https://edu-roman-kolyada.yc-sirius-dev.pelid.team)


### 1. Установите инструменты
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Lens Desktop](https://k8slens.dev/) (бесплатная версия для управления кластером)
- [Docker](https://www.docker.com/get-started/) (для сборки образа)

### 2. Получите доступ к кластеру
1. Войдите в [Yandex Cloud Console](https://console.cloud.yandex.ru).
2. Следуйте инструкциям в консоли для подключения к кластеру Kubernetes (yc-sirius-dev) через kubectl.
3. Установите Lens Desktop, подключите кластер, найдите namespace `edu-roman-kolyada`.

### 3. Создайте Secret для SSL PostgreSQL
Secret `postgres-ssl-cert` нужен для подключения к базе данных. Используйте `root.crt` из Yandex Cloud (или secret `postgres`).
```bash
kubectl create secret generic postgres-ssl-cert -n=edu-roman-kolyada --from-file=root.crt=<path_to_root.crt>
```

### 4. Создайте Secret для Django
Файл `deploy/yc-sirius-dev/edu-roman-kolyada/django-secret`.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: django-secret
  namespace: edu-roman-kolyada
type: Opaque
stringData:
  SECRET_KEY: "<generate-your-secret-key>"
```
Примените:
```bash
kubectl apply -f deploy/yc-sirius-dev/edu-roman-kolyada/django-secret.yml
```

### 5. Создайте ConfigMap для Django

Файл `deploy/yc-sirius-dev/edu-roman-kolyada/django-config`.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: django-config
  namespace: edu-roman-kolyada
data:
  DJANGO_SETTINGS_MODULE: "webapp.settings"
  ALLOWED_HOSTS: "127.0.0.1,localhost,edu-roman-kolyada.yc-sirius-dev.pelid.team"
```

Примените:
```bash
kubectl apply -f deploy/yc-sirius-dev/edu-roman-kolyada/django-config.yml
```

### 6. Настройте Nginx для reverse proxy
Файл `deploy/yc-sirius-dev/edu-roman-kolyada/main-nginx-config.yml` уже настроен для прокси на Django. 

Примените:
```bash
kubectl apply -f deploy/yc-sirius-dev/edu-roman-kolyada/main-nginx-config.yml
```
Перезапустите Nginx:
```bash
kubectl rollout restart deployment main-nginx -n edu-roman-kolyada
```

### 7. Соберите и опубликуйте Docker-образ

Соберите образ с тегом по git-хэшу:

```bash
docker build -t romigo24/django-site:$(git rev-parse --short HEAD) .
```

Опубликуйте:

```bash
docker push romigo24/django-site:$(git rev-parse --short HEAD)
```

### 8. Разверните Django
Файлы `deploy/yc-sirius-dev/edu-roman-kolyada/django-service.yml` и `django-deployment.yml`:
```bash
kubectl apply -f deploy/yc-sirius-dev/edu-roman-kolyada/django-service.yml
kubectl apply -f deploy/yc-sirius-dev/edu-roman-kolyada/django-deployment.yml
```
Обновите image в `django-deployment.yml` на новый git-хэш.

### 9. Настройте базу данных
Войдите в Django Pod:
```bash
kubectl exec -it $(kubectl get pod -n edu-roman-kolyada -l app=django -o name) -- bash
```
Выполните:
```bash
python manage.py migrate
python manage.py createsuperuser
```

### 10. Проверьте сайт

Сайт: `https://edu-roman-kolyada.yc-sirius-dev.pelid.team`

Админка: `https://edu-roman-kolyada.yc-sirius-dev.pelid.team/admin`

Логи: `kubectl logs -n edu-roman-kolyada -l app=django`

Статус Pod'ов: `kubectl get pods -n edu-roman-kolyada`