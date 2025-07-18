# Задание 5. Управление трафиком внутри кластера Kubernetes

[Ссылка](policies.yaml) на yaml файл с сетевыми политиками

Инлайном:

```yaml
# admin-api-allow
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: admin-api-allow
  namespace: prop-development
spec:
  podSelector:
    matchLabels:
      role: admin-back-end-api
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: admin-front-end
  egress:
    - to:
        - podSelector:
            matchLabels:
              role: admin-front-end


---

# non-admin-api-allow
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: non-admin-api-allow
  namespace: prop-development
spec:
  podSelector:
    matchLabels:
      role: back-end-api
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: front-end
  egress:
    - to:
        - podSelector:
            matchLabels:
              role: front-end
```

## Проверка политик.

При запущеном minikube:

Определили неймспейс

```bash
export NAMESPACE=prop-development
```

Создали неймспейс при необходимости
```bash
kubectl get namespace $NAMESPACE > /dev/null 2>&1 || kubectl create namespace $NAMESPACE
```

### Запуск сервисов

```bash
kubectl run front-end-app --image=nginx --labels role=front-end --namespace=$NAMESPACE --expose --port 80
kubectl run back-end-api-app --image=nginx --labels role=back-end-api --namespace=$NAMESPACE --expose --port 80
kubectl run admin-front-end-app --image=nginx --labels role=admin-front-end --namespace=$NAMESPACE --expose --port 80
kubectl run admin-back-end-api-app --image=nginx --labels role=admin-back-end-api --namespace=$NAMESPACE --expose --port 80
```

### Проверка запрещающей политики: 

```
$ kubectl run test-$RANDOM --rm -i -t --image=alpine --namespace=$NAMESPACE -- sh -c "wget -qO- --timeout=2 http://back-end-api-app" || echo "Connection failed as expected"

Connection failed as expected
```
### Проверка разрешающей политики:

```
$ kubectl exec -n $NAMESPACE -it front-end-app -- curl http://back-end-api-app

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```