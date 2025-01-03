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

## API

### LIST and WATCH filtering

### Set reference in API objects

## Using labels effectively

## Updating labels

