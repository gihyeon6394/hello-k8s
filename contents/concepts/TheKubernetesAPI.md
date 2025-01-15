# The Kubernetes API

- Discovery API
- OpenAPI interface definition
- Persistence
- API groups and versioning
- API Extension

Kubernetes API를 통해 k8s object를 쿼리하고, 조종
k8s control plane 중심은 API server로서 HTTP API를 제공하고, 사용자, 컴포넌트들은 API server를 통해 k8s와 상호작용함

- API server : HTTP API를 노출시켜 사용자, 다른 클러스터의 컴포넌트, 외부 컴포넌트들과 상호작용
- k8s API : k8s object를 생성, 수정, 삭제, 조회하는데 사용 e.g. Pods, Namespaces, ConfigMaps, ...
- kubectl CLI 혹은 kubeadm 같은 CLI 툴로 API server에 접근
    - REST 요청으로 직접 접근 가능

각 k8s 클러스터는 클러스터가 서빙하고있는 API를 명세하고, kubectl은 API 스펙을 가져와 캐싱하여 자동완성 등의 기능 제공

- Discovery API: k8s API에 대한 정보를 제공하는 API e.g. API name, resources, versions, ...
    - resource에 대한 상세 정보를 포함하지 않음
- Kubernetes OpenAPI Document : k8s API에 대한 스펙을 정의하는 문서
    - v3가 더 편리하고 자세해서 선호됨
    - 모든 API path 포함
    - 클러스터가 지원하는 확장 컴포넌트 명세

---

## Discovery API

각 리소스에대한 다음 내용을 포함

- Name
- Cluster or namespaced scope
- Endpoint URL and supported verbs- Alternative names
- Group, version, kind

### Aggregated discovery

`/api`, `/apis` endpoint를 통해 접근, 클러스터로부터 data를 fetch하기 위한 요청수를 줄일 수 있음  
`Accept` header를 통해 aggregated discover resource 지정 가능, 생략하면 응답으로 unaggregated resource 반환
e.g. `Accept: application/json;v=v2;g=apidiscovery.k8s.io;as=APIGroupDiscoveryList`

### Unaggregated discovery

`/api`, `/apis` 로 클러스터의 모든 그룹과 버전을 조회할 수 있음  
특정 그룹의 버전을 조회하기위해 추가적으로 `/apis/<group>/<version>` 요청 필요 e.g. `/apis/rbac.authorization.k8s.io/v1`

```json
{
  "kind": "APIGroupList",
  "apiVersion": "v1",
  "groups": [
    {
      "name": "apiregistration.k8s.io",
      "versions": [
        {
          "groupVersion": "apiregistration.k8s.io/v1",
          "version": "v1"
        }
      ],
      "preferredVersion": {
        "groupVersion": "apiregistration.k8s.io/v1",
        "version": "v1"
      }
    },
    {
      "name": "apps",
      "versions": [
        {
          "groupVersion": "apps/v1",
          "version": "v1"
        }
      ],
      "preferredVersion": {
        "groupVersion": "apps/v1",
        "version": "v1"
      }
    }
  ]
}
```

## OpenAPI interface definition

k8s 는 OpenAPI v2.0, 3.0을 지원하고 3이 선호됨

### OpenAPI v2

`/openapi/v2` 로 접근

| Header          | Possible values                                                                 | Notes                      |
|-----------------|---------------------------------------------------------------------------------|----------------------------|
| Accept-Encoding | gzip                                                                            | Optional                   |
| Accept          | application/json, application/com.github.proto-openapi.spec.v2@v1.0+protobuf, * | default : application/json |

### OpenAPI v3

`/openapi/v3` 로 접근

```json
{
  "paths": [
    {
      "api/v1": {
        "serverRelativeURL": "/openapi/v3/api/v1?hash=CC0E9BFD992D8C59AEC98A1E2336F899E8318D3CF4C68944C3DEC640AF5AB52D864AC50DAA8D145B3494F75FA3CFF939FCBDDA431DAD3CA79738B297795818CF"
      },
      "apis/admissionregistration.k8s.io/v1": {
        "serverRelativeURL": "/openapi/v3/apis/admissionregistration.k8s.io/v1?hash=E19CC93A116982CE5422FC42B590A8AFAD92CDE9AE4D59B5CAAD568F083AD07946E6CB5817531680BCE6E215C16973CD39003B0425F3477CFD854E89A9DB6597"
      }
    }
  ]
}
```

client-side cashing 가능 (`Expires`, `Cache-Control` header로 확인)

| Header          | Possible values                                                                 | Notes                      |
|-----------------|---------------------------------------------------------------------------------|----------------------------|
| Accept-Encoding | gzip                                                                            | Optional                   |
| Accept          | application/json, application/com.github.proto-openapi.spec.v2@v1.0+protobuf, * | default : application/json |

### Protobuf serialization

Protobuf 기반 직렬화 가능 (infra-cluster 통신에 유용)

## Persistence

k8s는 object의 상태를 직렬화하여 `etcd`에 저장함

## API groups and versioning

- 같은 resource에 대해 2개의 API 버전이 있다는 가정하에 `v1`, `v1beta1`
- `v1beta1` version API를 사용해 object를 생성했다면, 해당 object를 조작 (일기, 쓰기, 삭제)할떄는 `v1beta1`, `v1` version API 사용 가능
- `v1beta1` 이 deprecated 되면, `v1` version API를 사용해 object를 조작

k8s는 여러 API 경로를 지원 e.g. `/api/v1`, `/apis/rbac.authorization.k8s.io/v1alpha1`  
k8s 는 API groups를 구현하여 API를 확장 (disabled 가능), API resource는 API group, resource type, namespace, name으로 구분 가능  
모든 version은 같은 persisted data를 나타냄

### API changes

K9s API를 지속적으로 발전시키고 있음  
한번 GA(Genaral Availability)로 릴리즈된 API에 대한 호환을 보장 (`v1`)  
_beta_ API를 사용했다면, 해당 API가 deprecated 되었을때, 다른 API로 전환하면 됨 (deprecated API 상태일떄는 기존 GA API와 호환이 되는 시기)

## API Extension

k8s API는 2가지 방법으로 확장 가능

- 방법 1. custom resource : declarative하게 API server가 resource API를 제공하는 방법을 확장
- 방법 2. aggregation layer
