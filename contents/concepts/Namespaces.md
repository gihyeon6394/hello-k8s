# Namespaces

- When to Use Multiple Namespaces
- Initial namespaces
- Working with Namespaces
- Namespaces and DNS
- Not all objects are in a namespace
- Automatic labeling

_namespaces_ 는 한 클러스터 내에서 리소스를 그룹화하게 함.  
resource의 name은 namespace 안에서 unique해야하지만, 다른 namespace에서는 중복될 수 있음.

- namespaced object : Deployments, Services, ...
- cluster-wide objects : StorageClass, Nodes, PersistentVolumes, ...

---

## When to Use Multiple Namespaces

- 목적 : 많은 유저가 팀, 프로젝트 간에 존재할 때
    - 10명정도의 유저가 있다면, namespace를 사용하지 않아도 됨
- name의 범위를 제공함
    - resource의 name은 namespace 안에서 unique해야함 (다른 namespace에서는 중복될 수 있음)
    - k8s resource는 반드시 한개의 namespace에만 속할 수 있음
- 유저들간에 클러스터 리소스를 구분하게 해줌

## Initial namespaces

- **default** : 클러스터 생성시 자동으로 생성되는 namespace
- **kube-node-lease** : 각 노드별 Lease object 가 속함
    - Node lease는 kublet이 heartbeat를 보내는 역할
- **kube-public** : 모든 클라이언트에게 읽기 권한이 있는 namespace
    - public하게 공개되어져야하는 리소스가 속함
    - convention일 뿐, 필수 namespace는 아님
- **kube-system** : k8s 시스템 리소스가 속함

## Working with Namespaces

### Viewing namespaces

````
kubectl get namespace
NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
````

### Setting the namespace for a request

`--namespace` flag로 namespace를 지정

````
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>
kubectl get pods --namespace=<insert-namespace-name-here>
````

### Setting namespace preference

영구적으로 namespace를 설정하려면, `kubectl config set-context` 명령어 사용

````
kubectl config set-context --current --namespace=<insert-namespace-name-here>
# Validate it
kubectl config view --minify | grep namespace:
````

## Namespaces and DNS

Service를 생성할때 그에 따른 DNS entry가 생성됨 `<service-name>.<namespace-name>.svc.cluster.local`  
컨테이너가 `service-name`만으로 쿼리하면, 해당 namespace 내의 `service-name` 연결됨  
개발-스테이징-프로덕션에 걸쳐 같은 컨피그를 사용하는 경우 유용  
(다른 namespace로 연결하려면 FQDN 사용)

## Not all objects are in a namespace

대부분의 k8s resource는 namespace에 속함. 그러나 namespace 리소스는 namespace에 속하지 않음  
low-level resource (Nodes, persistentVolumes)는 namespace에 속하지 않음

````
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
````

## Automatic labeling

k8s control plane은 `NamespaceDefaultLabelName` 가 활성화된 경우  
자동으로 불변 label `kubernetes.io/metadata.name`을 모든 namespace에 추가함
