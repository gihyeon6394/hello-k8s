# Owners and Dependents

- Owner references in objects specifications
- Ownership and finalizers

k9s의 몇 Object는 다른 object의 _owner_ 가 될 수 있음 e.g. Replicaset 은 해당 pod 집합의 owner

---

## Owner references in objects specifications

- `metadata.ownerReferences` 필드를 가진 object는 Dependent object
    - 필드 값에 해당 object의 owner object를 가짐
    - 같은 namespace 내에 owner object name + UID
    - k8s는 자동으로 해당 필드에 값을 할당함
        - value를 바꿔서 수동으로 관계가 맺어지게 할 수 있음
            - 일반적인 방법은 아님
- `ownerReferences.blockOwnerDeletion` boolean 필드 : 종속객체 garbage collection이 owner object 삭제 가능 여부
    - default : true (`metadata.ownerReferences` 가 있을 시)

## Ownership and finalizers

- k8s에게 Resource 삭제 요청시 사전에 정의된 Finalizer rule에 따라 진행됨
    - 자동으로 설정된 finalizer (e.g. `kubernetes.io/pv-protection`) 는 의도치않게 resource가 삭제되는 것을 방지함
- foreground, orphan cascading deletion 시 k8s는 자동으로 owner resource에게 finalizer를 추가함
    - foreground deletion : `foreground` finalizer를 추가해 종속 resosource 삭제 후 owner resource 삭제
        - 종속 resource : `ownerReferences.blockOwnerDeletion=true` 인 object
    - orphan deletion policy : `orphan` finalizer를 추가해 owner object를 삭제 후 의존 resource는 무시함
