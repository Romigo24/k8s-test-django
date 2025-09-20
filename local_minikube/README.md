## Развертывание в Minikube

### 1. Установите инструменты
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm](https://helm.sh/docs/intro/install/)

### 2. Запустите Minikube
```sh
minikube start
```
### 3. Создайте Secret для Django
Файл `manifests/django-secret.yaml`:
```yml
apiVersion: v1
kind: Secret
metadata:
  name: django-secrets
type: Opaque
data:
  secret-key: $(echo -n "your-secret-key" | base64)
  database-url: $(echo -n "postgres://user:password@host:5432/db" | base64)
```
Примените:
```bash
kubectl apply -f manifests/django-secret.yaml
```

### 4. Разверните PostgreSQL в кластере
```bash
helm install postgresql oci://registry-1.docker.io/bitnamicharts/postgresql   --version 15.5.20   --set auth.username=test_user   --set auth.password=test_password   --set auth.database=test_db
```


### 5. Примените манифесты Django
```bash
kubectl apply -f manifests/deployment.yml
kubectl apply -f manifests/service.yml
kubectl apply -f manifests/ingress.yml
kubectl apply -f manifests/clearsessions-cronjob.yml
kubectl apply -f manifests/migrate-job.yml
```

### 6. Проверьте сайт
После настройки `/etc/hosts` сайт будет доступен по адресу:
[http://star-burger.test](http://star-burger.test)

---