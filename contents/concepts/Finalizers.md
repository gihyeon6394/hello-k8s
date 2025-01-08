# Finalizers

- How finalizers work
- Owner references, labels, and finalizers

finalizer 는 resource가 완전히 삭제되기 전에 반드시 만족해야하는 조건을 k8s 에 정의하는 namespaced-key  
GC 처럼 사용가능 (e.g. controller가 object 삭제 전에 연관된 Infra resource를 해제하도록 지시)
액션은 보통 코드로 정의하기보다는 애토에시녀과 같은 key로 정의

1. finalizer가 정의된 object를 삭제할때는
2. k8s API가 `.metadata.deletionTimestamp` 를 object에 붙여 삭제하도록 지시하고, `202` status code를 반환
3. target objectsms terminating state에 남음
4. control palne, other component 들이 finalizer를 만족시키기 위해 작업을 수행
5. finalizer가 만족되면, object로부터 Finalizer를 제거함
6. `metadata.finalizers` 가 비었으면 k8s는 deletion 작업이 완료된것으로 간주하고, object를 삭제함

---

## How finalizers work

manifest 의 `metadata.finalizers` 필드에 finalizer를 추가하면, object가 삭제되기 전에 finalizer를 만족시키기 위해 작업을 수행해야함을 나타냄

1. `metadata.finalizers` 가 있는 object 삭제 시도
2. API server가 `metadata.deletionTimestamp` 를 object에 추가하고, 삭제 시작 시간을 초기화
    - object deletion이 요청되었음을 나타냄
3. `metadata.finalizers` 에 있는 모든 아이템이 사라질 때까지 object는 삭제되지 않음
    - controller는 Finalizer를 조건이 만족되는 지 계속 체크하며 만족할떄마다다 `finalizers` 필드에서 key 제거
    - 모든 key가 사라지면, `deletionTimestamp` 가 설정된 object는 자동으로 삭제됨
4. return `202` status code (Accepted)

#### usage example : `kubernetes.io/pv-protection`

- `PersistentVolume` object가 의도치않게 삭제되는 것을 방지하기 위한 finalizer
- Pod가 `PersistentVolume` object를 사용하고 있을때, k8s 는 `pv-protection` finalizer를 추가함
- 해당 PersistentVolume을 삭제시도 시
    - `Terminating` status로 변경되고, finalizer가 있는한 삭제될 수 없음
- pod가 `PersistentVolume`을 사용을 안하면, k8s는 `pv-protection` finalizer를 제거하고, `PersistentVolume`을 삭제함

#### 주의

- `DELETE` object 시, k8s는 deletion timestamp를 object에 추가하고, 즉시 `.metadata.finalizers` 필드 수정을 막음
    - 즉, pending deletion 상태일때 `.metadata.finalizers` 필드에 key 제거만 가능 (추가 불가)
    - `deletionTimestamp`도 한번 설정되면 값을 변경할 수 없음
- deletion 요청이 들어오면, 해당 Object를 부활시킬 수 없음

## Owner references, labels, and finalizers

label 처럼 owner reference 는 object 간의 관계를 정의하는데 사용됨 (목적은 다름)  
controller는 label로 연관 object의 변경을 추적함 e.g. `Job`이 1개 이상의 pod를 생성할 때, job controller는 해당
Pod들에 label을 추가해, 동일 클러스터 내에 해당 label을 가진 pod를 추적함

Job controller는 해당 Pod에 _owner reference_ 를 추가함 (pod를 생성한 Job을 가리킴)  
Pod가 실행중인데 Job을 삭제하면, k8s는 _owner reference_ 를 통해 클러스터 내에 어떤 pod가 삭제되어야 하는지 추적함

삭제하려는 resource가 owner reference가 있을때 finalizer를 실행함  
어떤 경우, 의존 object의 삭제를 blocking 하여, owner가 오랜시간 삭제되지 않고 있을 수 있음

#### 주의

object가 deleting state에 남아있다면, finalizer를 수동으로 제거하는 것을 지양할 것
finalizer가 이유가 있어서 붙어있을것이므로 강제로 제거하면 cluster의 상태가 불안정해질 수 있음
