# Object Names and IDs

- Names
- UIDs

클러스터의 각 Object는 _Name_, _UID_ 가짐.

- Name : 해당 Resource type 내에서 고유한 이름
    - e.g. 동일 namespace 안에서 `myapp-1234` name인 pod는 하나만 존재
    - `myap-1234` 라는 pod, `myapp-1234` 라는 development는 각자 하나씩 존재 가능
- UID : 전체 클러스터 안에서 고유한 ID
- labels, annotations : object를 식별하는 데 사용 가능 (not unique)

---

## Names

resouce URL에서 object를 가리키는 식별자 e.g. `/api/v1/pods/some-name`  
동시에 같은 종류의 Object 안에서 고유해야함 (해당 object를 delete하면 name 재사용 가능)
`generateName` 이 있으면 자동으로 `name`을 생성 (prefix로 사용), 생성시 중복발생하면 HTTP 409 응답  
리소스 naming에 사용되는 제약조건은 4가지 (DNS subdomain, RFC 1123 label, RFC 1035 label, path segment)

- 주의 : Name은 동일한 resource에 대해 모든 API Version에 걸쳐 고유해야함
- API resource는 버전에 의해 구별되지 않음
    - API group, resource type, namespace, name으로 구별됨

### DNS Subdomain Names

- 253 characters 이하
- 소문자의 알파벳, 숫자, `-`, `.` 만 사용 가능
- 알파벳, 숫자로 시작
- 알파벳, 숫자로 끝

### RFC 1123 Label Names

- 63 characters 이하
- 소문자의 알파벳, 숫자, `-` 만 사용 가능
- 알파벳, 숫자로 시작
- 알파벳, 숫자로 끝

### RFC 1035 Label Names

- 63 characters 이하
- 소문자의 알파벳, 숫자, `-` 만 사용 가능
- 알파벳 시작
- 알파벳, 숫자로 끝

### Path Segment Names

- path segment 로 안전하게 인코딩되기 위함
- `.`, `..`이 될 수 없음
- `/`, `%` 사용 불가

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80

```

## UIDs

k8s system-generated unique identifier  
k8s cluster 전체 생명주기에 걸쳐 unique
