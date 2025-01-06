# Labels and Selectors

- Motivation
- Syntax and character set
- Label selectors
- API
- Using labels effectively
- Updating labels

_lables_ 는 key-value 쌍으로 object에 첨부 가능, 생성시점, 그리고 그 이후에도 수정 가능  
각 key는 해당 Object에서 유일해야함

```
"metadata": {
  "labels": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```

---

## Motivation

- `"release" : "stable"`, `"release" : "canary"`
- `"environment" : "dev"`, `"environment" : "qa"`, `"environment" : "production"`
- `"tier" : "frontend",` `"tier" : "backend"`, `"tier" : "cache"`

위처럼 label을 사용하면, object를 선택하고, group, filter 가능  
Service 배포, 배치 프로세스 파이프라인은 multi-dimensional entity (multiple release, environment, tier)를 다루어야함

## Syntax and character set

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production # production 환경
    app: nginx # nginx 앱
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
```

- label key는 2개의 segment로 구성
    - optional prefix + `/` + required name
    - 최대 63 characters
    - 시작과 끝이 알파벳, 숫자
    - 중간에는 `-`, `_`, `.`, 알파펫, 숫자 사용 가능
- name
    - 최대 63 characters
    - 알파벳, 숫자, `-`, `_`, `.` 사용 가능
- prefix
    - 있다면 반드시 DNS subdomain 형태여야함
- `kubernetes.io/`, `k8s.io/` prefix는 reserved

## Label selectors

- 유의 : names, UID와 달리 label은 unique하지 않음

_label selector_로 object 집합 선택 가능, 2가지 타입의 selector 존재 (_equality-based_, _set-based_)  
comma-separated 형태로 여러 조건 사용 가능 (AND 조건)

### Equality-based requirement

````
# environment가 production 인 resource 선택 
environment = production

# tier가 frontend가 아닌 resource 선택
tier != frontend

# environment가 production이고, tier가 frontend가 아지 않은 resource 선택
environment=production,tier!=frontend
````

- label의 key와 value가 일치하는 object 선택
- 선택하려는 key-value가 있지만 추가적인 label 존재해도 선택됨
- `=`, `==`, `!=` 사용 가능

### Set-based requirement

````
# environment가 production이거나 qa인 resource 선택
environment in (production, qa)

# tier가 frontend가 아닌 resource 선택
tier notin (frontend, backend)

# label key로 partition이 있는 resource 선택
partition

# label key로 partition이 없는 resource 선택
!partition

# partition이 있고, environment가 qa가 아닌 resource 선택
partition,environment notin (qa)

# partition이 customerA 또는 customerB이고, environment가 qa가 아지 않은 resource 선택
partition in (customerA, customerB),environment!=qa
````

- `in`, `notin`, `exists` 사용 가능

## API

### LIST and WATCH filtering

- `list`, `watch` operation
- query parameter로 필터 추가
    - equality-based requirements: `?labelSelector=environment%3Dproduction,tier%3Dfrontend`
    - set-based requirements: `?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`

```
# apiserver에게 kubectl을 사용해 equality-based requirements 전달
kubectl get pods -l environment=production,tier=frontend

# set-based requirements 전달
kubectl get pods -l 'environment in (production),tier in (frontend)'
kubectl get pods -l 'environment in (production, qa)'
kubectl get pods -l 'environment,environment notin (frontend)'
```

### Set reference in API objects

`service`, `replicationcontrollers` 같은 object는 pods를 선택하기 위해 label selector 사용

#### Service and ReplicationController

`component=redis`, `component in (redis)` 같은 label selector 사용

```yaml
"selector": {
  "component": "redis",
}

# or
selector:
  component: redis
```

#### Resources that support set-based requirements

```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - { key: tier, operator: In, values: [ cache ] }
    - { key: environment, operator: NotIn, values: [ dev ] }

```

- `matchLabels` : key-value 쌍 맵
- `matchExpressions` : pod selector requirements 집합
    - `key` : label key
    - `operator` : `In`, `NotIn`, `Exists`, `DoesNotExist`
    - `values` : label value 집합

#### Selecting sets of nodes

- 파드가 스케쥴링 될 노드를 제한하기 위해 label selector 사용 가능

## Using labels effectively

- 리소스 간에 구분, 선택을 위해 label 사용

guestbook app의 fe

```yaml
labels:
  app: guestbook
  tier: frontend
```

guestbook app, redis master

```yaml
labels:
  app: guestbook
  tier: backend
  role: master
```

guestbook app, redis replica

```yaml
labels:
  app: guestbook
  tier: backend
  role: replica
```

````
# guestbook app 전체 배포
kubectl apply -f examples/guestbook/all-in-one/guestbook-all-in-one.yaml

# label에 app, tier, role이 있는 pod 선택
kubectl get pods -Lapp -Ltier -Lrole
NAME                           READY  STATUS    RESTARTS   AGE   APP         TIER       ROLE
guestbook-fe-4nlpb             1/1    Running   0          1m    guestbook   frontend   <none>
guestbook-fe-ght6d             1/1    Running   0          1m    guestbook   frontend   <none>
guestbook-fe-jpy62             1/1    Running   0          1m    guestbook   frontend   <none>
guestbook-redis-master-5pg3b   1/1    Running   0          1m    guestbook   backend    master
guestbook-redis-replica-2q2yf  1/1    Running   0          1m    guestbook   backend    replica
guestbook-redis-replica-qgazl  1/1    Running   0          1m    guestbook   backend    replica
my-nginx-divi2                 1/1    Running   0          29m   nginx       <none>     <none>
my-nginx-o0ef1                 1/1    Running   0          29m   nginx       <none>     <none>

# app=guestbook, role=replica인 pod 선택
kubectl get pods -lapp=guestbook,role=replica
NAME                           READY  STATUS   RESTARTS  AGE
guestbook-redis-replica-2q2yf  1/1    Running  0         3m
guestbook-redis-replica-qgazl  1/1    Running  0         3m
````

## Updating labels

- `kubectl label` command로 label 추가, 수정, 삭제 가능

````
# nginx app에 tier=fe label 추가
kubectl label pods -l app=nginx tier=fe
pod/my-nginx-2035384211-j5fhi labeled
pod/my-nginx-2035384211-u2c7e labeled
pod/my-nginx-2035384211-u3t6x labeled

# nginx app이면서 tier label이 있는 pod 선택
kubectl get pods -l app=nginx -L tier
NAME                        READY     STATUS    RESTARTS   AGE       TIER
my-nginx-2035384211-j5fhi   1/1       Running   0          23m       fe
my-nginx-2035384211-u2c7e   1/1       Running   0          23m       fe
my-nginx-2035384211-u3t6x   1/1       Running   0          23m       fe
````

