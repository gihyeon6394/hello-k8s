# Recommended Labels

- Labels
- Applications And Instances Of Applications
- Examples

k8s Object를 대시보드나 kubectl로 시각화 가능하고, 이때 라벨이 유용
recommended label 이 있을 뿐 필수는 아님

`app.kubernetes.io` : 공용 라벨, 어노테이션이 사용하는 prefix

---

## Labels

```yaml
# This is an excerpt
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxyz
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
    app.kubernetes.io/managed-by: Helm

```

아래 라벨들을 모든 resource object에 붙여야함

| Key                            | Example      | Type   | Description              |
|--------------------------------|--------------|--------|--------------------------|
| `app.kubernetes.io/name`       | `nginx`      | string | 애플리케이션 이름                |
| `app.kubernetes.io/instance`   | `my-nginx`   | string | 애플리케이션 인스턴스 식별자 (unique) |
| `app.kubernetes.io/version`    | `1.0.0`      | string | 애플리케이션 버전                |
| `app.kubernetes.io/component`  | `web-server` | string | 애플리케이션 컴포넌트 (아키텍처)       |
| `app.kubernetes.io/part-of`    | `wordpress`  | string | 한수준 위 애플리케이션 이름          |
| `app.kubernetes.io/managed-by` | `Helm`       | string | 애플리케이션 관리 툴              |

## Applications And Instances Of Applications

애플리케이션은 k8s cluster 내에서 한번 이상 설치 가능 (같은 namespace 내에서도) e.g. `wordpress` application은 여러개의 인스턴스를 가질 수 있음  
application 이름 (name)과 인스턴스 이름은 독립적으로 기록됨

- `app.kubernetes.io/name` : 애플리케이션 이름 e.g. `wordpress`
- `app.kubernetes.io/instance` : 애플리케이션 인스턴스 이름 e.g. `wordpress-a`, `wordpress-b`, ...
- 모든 애플리케이션의 인스턴스명은 unique해야함

## Examples

### A Simple Stateless Service

`Deployment`, `Service` object를 생성하고 라벨 붙이기

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: myservice
    app.kubernetes.io/instance: myservice-abcxyz
# ...
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: myservice
    app.kubernetes.io/instance: myservice-abcxyz
# ...
```

### Web Application With A Database

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxyz
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxyz
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
```

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxyz
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxyz
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
```
