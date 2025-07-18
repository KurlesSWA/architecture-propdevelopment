# Задание 4. Защита доступа к кластеру Kubernetes

## Таблица ролей и полномочий в Kubernetes

| Роль            | Права роли                                                                                                                               | Группы пользователей                        |
| :-------------- |:-----------------------------------------------------------------------------------------------------------------------------------------| :------------------------------------------ |
| `cluster-admin` | Встроенная роль с полным доступом ко всем ресурсам во всех пространствах имен (verb: `*`, resource: `*`, apiGroup: `*`).                 | DevOps-инженеры, администраторы кластера.   |
| `developer`     | Создание, обновление, удаление и просмотр рабочих нагрузок (pods, deployments, services, ingresses) в пределах своего пространства имен. | Разработчики                                |
| `auditor`       | Просмотр всех ресурсов, включая секреты (secrets).                                                                                       | Специалисты по информационной безопасности. |
| `viewer`        | Просмотр неконфиденциальных ресурсов (pods, services, deployments).                                                                      | Операционная команда.                       |

## Создание ролей

[Ссылка](create_roles.yaml) на yaml файл

Оно же инлайном:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: prop-development
  name: developer
rules:
- apiGroups: ["", "apps", "extensions"]
  resources: ["pods", "deployments", "services", "ingresses"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: auditor
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: prop-development
  name: viewer
rules:
- apiGroups: ["", "apps", "extensions"]
  resources: ["pods", "services", "deployments"]
  verbs: ["get", "list", "watch"]
```

## Создание пользователей

[Ссылка](create_users.yaml) на yaml файл

Оно же инлайном:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: developer
  namespace: propdev
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: operator
  namespace: propdev
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: auditor
  namespace: propdev
```

## Привязка ролей

[Ссылка](bind_roles.yaml) на yaml файл

Оно же инлайном:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
  namespace: prop-development
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: audit-user-binding
subjects:
- kind: User
  name: audit-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: auditor
  apiGroup: rbac.authorization.k8s.io
```