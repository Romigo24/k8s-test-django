## Развертывание в Yandex Cloud (Kubernetes кластер yc-sirius-dev)

### Инструкции для деплоя Django-сайта в dev-окружении Kubernetes (`namespace: edu-roman-kolyada`). 


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