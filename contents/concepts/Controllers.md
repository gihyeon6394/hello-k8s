# Controllers

- Controller pattern
- Desired versus current state
- Design
- Ways of running controllers

_control loop_ 란 로봇공학에서 사용하는 용어로 시스템의 상태를 조절하는 무한루프를 말함 (e.g. 방의 온도 조절 장치)  
방의 온도를 장치를 통해 설정하면 방의 _desired state_ 가 설정되고, 현재 온도가 _current state_ 로 설정됨. 장치는 _current state_ 와 _desired state_ 를 비교하여
_current state_ 가 _desired state_ 와 일치하도록 조절함. 이러한 루프를 _control loop_ 라고 함.  
k8s의 controller는 control loop로서 cluster의 state를 추적하고 필요한 조치를 함 (current state -> desired state)

---

## Controller pattern

controller는 한번 이상 k8s resource type을 추적함. 해당 object들은 spec field에 _desired state_ 를 가지고 있음. controller는 object의 _current
state_ 를 추적하고 _desired state_ 와 비교하여 _desired state_ 가 되도록 조치함.

### Control via API server (e.g. Job controller)

- desired state가 될수 있게 API server에 직접 요청
- object에 대한 설정을 업데이트

* Job : 1개 이상의 Pod에서 실행되는 K8s resource, task 실행 후 정지. Job의 desired state는 완료 (completetion

k8s build-in controller (e.g. Job controller) 는 cluster API server화 통신하여 상태를 관리함  
job controller가 새로운 task를 발견하면, node의 kublet이 적정 수의 파드를 실행하고 종료할수 있게 함.   
Job controller가 직접 파드를 생성하거나 실행하지 않는 대신, API server에 파드를 생성/삭제 요청함  
Job controller는 job이 desired state (completetion)에 가까워지도록 job을 실행할 파드를 생성하도록 하여 job을 완료시킴

job이 완료되면 설정을 `Finished`로 변경

### Direct control (IP address management tool, storage service, cloud provider APIs)

k8s control plane은 간접적으로 IP address management tool, storage service, cloud provider APIs와 통신하여 cluster 외부 요인을 수정할 수 있음

cluster 외부요인을 수정해야하는 controller도 존재 (e.g. cluster 외부에 node 증설 알람을 요청하는 controller)  
API server로 부터 desired state를 받아 외부 시스템과 직접 통신하여 current state를 desired state로 변경 후 API server에 current state를 알림
state 변경이 됨을 API server에 알리고 해당 API server를 통해 다른 액션을 취할 수 있음

## Desired versus current state

클러스터는 언제든지 변할수있고 control loop은 자동으로 클러스터를 원하는 상태로 유지하려고 노력함 (**결국 desired state가 되지 않을수도 있음**)

## Design

k8s는 클러스터의 상태 제어를 위해 여러 컨트롤러를 관리하고, 각 컨트롤러는 클러스터의 특정 상태를 제어함.   
일반적으로 controller는 하나의 리소스를 desired state로 사용하고, 다른 리소스 타입을 사용하여 desired state가 되게 함
예를 들어, job을 관리하는 컨트롤러는 job obejct를 추적하고 (새로운 Job work를 발견하기 위함), pod object들도 추적함 (job을 실행하고, 완료됨을 알기 위함)

같은 리소스 타입의 오브젝트를 여러 컨트롤러에서 생성할지라도, 컨트롤러는 자신과 연결된 리소스에만 관여를 함.  
예를 들어, Deployments, Job 컨트롤러가 각자 pod를 생성할수 있지만, Job 컨트롤러는 Deployment가 생성한 파드를 생성하거나 관여하지 않음  
각 컨트롤러는 라벨에 관리가능한 파드 정보를 가지고 있음

## Ways of running controllers

k8s 는 kube-controller-manager 안에서 built-in으로 동작하는 여러 컨트롤러로 구성되어있음. built-in 컨트롤러는 여러 중요 기능을 제공함  
Deploy, Job 컨트롤러는 control plane을 탄력적으로 운용하여 built-in 컨트롤러중 fail이 발생하면 다른 파트의 컨트롤러가 해당 컨트롤러의 역할을 대신함  
k8s를 확장하기 위해 control plane 외부에 컨트롤러를 두거나, 새로운 컨트롤러를 생성 가능.

