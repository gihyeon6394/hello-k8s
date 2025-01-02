# Kubernetes Object Management

- Management techniques
- Imperative commands
- Imperative object configuration
- Declarative object configuration

`kubectl` CLI 툴로 k8s object를 생성, 수정, 삭제 가능

---

## Management techniques

| Management technique             | Operates on          | Recommended use      | Supported writers | Learning curve |
|----------------------------------|----------------------|----------------------|-------------------|----------------|
| Imperative commands              | Live objects         | Development projects | 1+                | Lowest         |
| Imperative object configuration  | Individual files     | Production projects  | 1                 | Moderate       |
| Declarative object configuration | Directories or files | Production projects  | 1+                | Highest        |        

## Imperative commands

- 사용자가 클러스터의 live object를 직접 조작하는 방법
- `kubectl` argument, flag 사용
- 주의 : 히스토리를 추적하기 어려움

### Examples

```
## development object 생성 + nginx container instance 실행
kubectl create deployment nginx --image nginx
```

### Trade-offs

- pros
    - single action word
      -cluster에 single step으로 object 조작
- cons
    - change review process X
    - audit trail X
    - template 미지원

## Imperative object configuration

operation + optional flags + 1개 이상의 file  
file은 YAML, JSON 형식으로 object를 full로 정의

### Examples

```
## nginx.yaml 로 object 생성
kubectl create -f nginx.yaml

## nginx.yaml 로 object 삭제, redis.yaml 로 object 생성
kubectl delete -f nginx.yaml -f redis.yaml

## nginx.yaml 로 object 변경 (overwrite)
kubectl replace -f nginx.yaml
```

### Trade-offs

- pros
    - GIT과 같은 VCS에 저장 가능
    - review process 가능
    - template 사용 가능
- cons
    - object schema에 대한 기본 이해로 파일 작성해야함
    - 파일 작성 필요
    - object를 수정할때는 반드시 파일이 수정되어야함
        - 파일 수정 없이 업데이트하면, 다음 업데이트시 변경된 사항이 반영되지 않음

## Declarative object configuration

file에 operation 까지 포함
object마다 서로 다른 operation을 수행할 수 있음

### Examples

```
## configs 디렉토리에 있는 모든 설정 파일과 live object 비교
kubectl diff -f configs/

## live object를 configs 디렉토리에 있는 설정 파일로 변경
kubectl apply -f configs/
```

```
## Recursive 옵션
kubectl diff -R -f configs/
kubectl apply -R -f configs/
```

### Trade-offs

- pros
    - live object에 직접 반영된 변경사항은 설정파일에 반영되지 않더라도 유지됨
    - directory 기반으로 operate 가능, object별 operation 자동 감지
- cons
    - debug 어려움
    - partial update 시 merge, patch operation이 복잡해짐
