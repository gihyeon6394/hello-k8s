# Annotations

- Attaching metadata to objects
- Syntax and character set

k8s annotation을 사용해 임의의 메타데이터를 object에 추가하여, 조회 가능
외부 데이터베이스, 디렉터리에 저장해도 되지만, 공유하기 번거로움

---

## Attaching metadata to objects

````yaml
"metadata": {
  "annotations": {
    "key1": "value1",
    "key2": "value2"
  }
}
````

label이나 annotation을 사용해 k8s object에 메타데이터를 추가할 수 있음

- Label : object를 선택, 특정 컨디션을 만족하는 object collection 검색에 사용
- Annotation : Object에 대한 추가정보로, 식별 (필터)하는 정보가 아님
    - Label에서 불가능한 특수문자도 가능
    - Key, value는 반드시 문자열이어야함
- 한 object에서 Label, Annotation 모두 사용 가능

#### annotation 정보 예시

- declarative config yaml로 관리되는 필드
- build, release, image 정보, timestamp, release ID 등
- 리퍼지터리 포인터 : Logging, monitoring, analytics
- 디버깅을 위한 클라이언트 라이브러리, 툴 정보 e.g. name, version, build info
- 경량 롤아웃 도구 메타데이터 e.g. config, checkpoints
- 기술 책임자 연락처, 팀 웹사이트 등
- 사용자 지시사항

## Syntax and character set

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotations-demo
  annotations:
    imageregistry: "https://hub.docker.com/"
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
```

- annotation은 key, value 쌍
- Key : optional prefix + `/` + name
    - optional prefix : 지정한다면 반드시 DNS subdomain 이어야함
        - 지정하지 않으면 해당 key를 prviate으로 취급
        - 자동으로 생성되는 system components (e.g. kube-scheduler)는 prefix를 사용
        - reserved : `kubernetes.io/`, `k8s.io/`
    - name : 63자 이하, 소문자, 숫자, `-`, `_`, `.`
        - 알파벳,숫자로 시작 끝
