# Field Selectors

- Supported fields
- Supported operators
- Chained selectors
- Multiple resource types

_Field selecotrs_ 로 k8s object를 선택 가능하다.

- `metadata.name=my-service`
- `metadata.namespace!=default`
- `status.phase=Pending`

````
# status.phase 필드가 Running 인 pod를 선택
kubectl get pods --field-selector status.phase=Running
````

---

## Supported fields

k8s resource type별로 지원하는 field selector가 다르다. (미지원 필드로 필터링하면 에러가 발생)  
모든 리소스 타입이 `metadata.name` 과 `metadata.namespace` 를 지원한다.

````
# 리소스타입 ingress 중 foo.bar 필드가 bax인 object 필터링 
kubectl get ingress --field-selector foo.bar=baz
Error from server (BadRequest): Unable to find "ingresses" that match label selector "", field selector "foo.bar=baz": "foo.bar" is not a known field selector: only "metadata.name", "metadata.namespace"
````

### List of supported fields

https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/#list-of-supported-fields

### Custom resources fields

`CustomResourceDefinition` 을 통해 `spec.versions[*].selectableFileds` 를 custom resource에서 사용할 필드 selector 지정 가능하다.

## Supported operators

`=`, `==`, `!=` 가능

```
# 리소스 타입 services and 모든 namespace and metadata.namespace 필드가 default가 아닌 object
kubectl get services  --all-namespaces --field-selector metadata.namespace!=default
```

## Chained selectors

`,` 로 구분하여 selector를 연결 가능하다.

````
# 리소스 타입 pod and status.phase 필드가 Running이 아님 and spec.restartPolicy 필드가 Always가 아닌 object
kubectl get pods --field-selector=status.phase!=Running,spec.restartPolicy=Always
````

## Multiple resource types

여러 Resource Type을 한번에 조회 가능하다.

```
# 리소스 타입 statefulsets or services and 모든 namespace and metadata.namespace 필드가 default가 아닌 object
kubectl get statefulsets,services --all-namespaces --field-selector metadata.namespace!=default
```
