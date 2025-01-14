## Objects In Kubernetes

- Understanding Kubernetes objects
- Server side field validation

k8s object는 k8s system과 cluster의 상태를 표현하기 위한 위한 영구 엔티티, `.yaml` 포맷으로 표현

- [Kubernetes Object Management](KubernetesObjectManagement.md)
- [Object Names and IDs](ObjectNamesAndIDs.md)
- [Labels and Selectors](LabelsAndSelectors.md)
- [Namespaces](Namespaces.md)
- [Annotations](Annotations.md)
- [Field Selectors](FieldSelectors.md)
- [Finalizers](Finalizers.md)
- [Owners and Dependents](OwnersAndDependents.md)
- [Recommended Labels](RecommendedLabels.md)

---

## Understanding Kubernetes objects

_kubernetes object_ 는 k8s cluster에서 실행되는 애플리케이션, 워크로드, 클러스터 상태를 표현하는 엔티티  
object가 생성되고 나면, k8s system은 object를 클러스터의 _desired state_에 맞게 유지하도록 함  
k8s objects는 K8s API로 생성/수정/삭제 `kubectl` command

- 각 노드에 어떤 application이 실행 중인지
- application이 사용가능한 리소스
- application 정책 e.g. 재시작, fault-tolerance

### Object spec and status

`spec`, `status` 오브젝트 필드를 가짐

- `spec` : object의 원하는 상태를 정의하는 필드, _desired state_
- `status` : object의 현재 상태를 나타내는 필드, _current state_
    - k8s system에 의해 업데이트됨
    - control plane이 지속적으로 모든 오브젝트의 `status`를 확인 -> _desired state_ 에 맞추려함

#### 예시 : Deployment object

- Deployment object : application이 cluster에서 실행됨을 나타냄
- Deployment object의 `spec` 필드 : application이 3개 실행됨을 설정

1. 3개 중 하나가 다운되면, `status` 필드가 2개로 업데이트됨
2. k8s system은 `status` 필드를 확인하고, `spec` 필드에 맞게 3개로 업데이트

### Describing a Kubernetes object

- 방법 1. k8s api로  (e.g. `kubectl` command)
    - 생성시 정보를 JSON으로 request body에 표현해야함
- 방법 2. _manifest_ 파일로 `kubectl` command에 전달
    - 컨벤션으로 파일은 YAML 포맷으로 작성
    - `kubectl` 은 YAML -> JSON 변환 후 API로 전달

    1. `kubectl apply -f https://k8s.io/examples/application/deployment.yaml`
    2. `deployment.apps/nginx-deployment created`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec: # desired state + basic infos
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

### Required fields

- `apiVersion` : object 생성에 사용할 k8s API 버전
- `kind` : object의 종류 (e.g. Deployment, Job, DaemonSet)
- `metadata` : object 식별 데이터 `name`, `UID`, `namespace` 등
- `spec` : object의 원하는 상태를 정의하는 필드

## Server side field validation

- v1.25 부터 API server에서 object 생성시 필드 유효성 검사
- `kubectl --validate` flag로 벨리데이션 레벨 설정 가능
    - `ignore` (false), `warn`, `strict` (true)
    - default : `Kubectl --validate=true` (strict)
- `strict` : 벨리데이션 실패시 에러
- `warn` : 벨리데이션 실패시 경고
- `ignore` : 벨리데이션 미수행

