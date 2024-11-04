# k8s-3

# Deployment 옵션

minikube는 로컬 머신의 가상머신에 더미 클러스터를 제공한다. 이러한 리소스를 제공하는 것은 쿠버네티스가 하는일이 아니다.

실제 배포를 할땐 쿠버네티스가 실행될 인프라를 설정해야한다. 쿠버네티스는 인프라를 설정해주지 않는다. 

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/621504a8-73f4-4a14-962c-b3160a44e406/26376d21-9adf-40c6-a627-86eab93c7305/image.png)

자신의 데이터 센터를 사용한다면 모든 것을 직접 구성해야한다.

클라우드 프로바이더를 사용한다면 두 가지 선택지가 있다.

1. 클라우드 프로바이더를 통하여 자체 가상 인스턴스(EC2), 자체 머신 가동. 이러한 머신에서 모든 소프트웨어 설치, 구성. (kops를 사용하여 좀 더 쉽게 리소스 구성가능)
2. 관리형 서비스인 EKS 사용. 일반적인 클러스터 아키텍처만 정의하면 서비스가 대신 구성해준다.

# EKS vs ECS

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/621504a8-73f4-4a14-962c-b3160a44e406/6d3e5bec-fe47-43f5-afe7-d11d6f7da50f/image.png)

**Kubernetes 배포를 위한 관리형 서비스 (AWS EKS)**

- **AWS 특정 구문이나 철학이 필요 없음**
- **표준 Kubernetes 구성 및 리소스 사용**

**컨테이너 배포를 위한 관리형 서비스 (AWS ECS)**

- **AWS 특정 구문 및 철학이 적용됨**
- **AWS 특정 구성 및 개념 사용**

쿠버네티스 지식을 활용하려면 EKS가 나은 선택이다.

여기서는 mongo atlas로 DB를 만들고 시작했다.

# EKS 클러스터 생성

https://kschoi728.tistory.com/81

EKS는 내부적으로 EC2를 만들어 진행한다. 

EKS가 나 대신 적절한 권한을 가지고 리소스를 만들려면 권한을 부여해야한다. IAM으로 설정 가능하다. EKS를 선택하면, EKS에 부여하도록 사전 정의된 역할이 제공된다.

네트워크 구성은 외부에서 접근과 내부에서 접근으로 구성되어야한다. 

네트워크를 생성하기 위해 [VPC](https://www.notion.so/Docker-10597d6ce88f8008895bf2885d37d508?pvs=21) 콘솔을 사용할 수 있다. CloudFormation을 이용해서 만든다. CloudFormation은 특정 템플릿 기반으로 쉽게 리소스를 만들 수 있게 도와준다.

[https://medium.com/pplink/aws-cloudformation으로-인프라-자동화-시작하기-9fe13cdf08c9](https://medium.com/pplink/aws-cloudformation%EC%9C%BC%EB%A1%9C-%EC%9D%B8%ED%94%84%EB%9D%BC-%EC%9E%90%EB%8F%99%ED%99%94-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-9fe13cdf08c9)

해당 url에서 s3 url을 찾아 복사한다. 생성하려는 네트워크에 대한 템플릿이 포함되어 있다.

https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml

스택 이름만 추가하고 생성.

만든 vpc와 클러스터 엔드 포인트를 퍼블릭, 프라이빗 둘 다로 설정

추가 로깅 필요하면 활성화 가능

EKS 클러스터 생성완료.

kubectl 명령어로 클러스터에 명령을 보냈다. 이제는 minnikube가 아닌 EKS 클러스터로 명령을 보내보자.

# kubectl은 minikube에게 명령을 보낸다는것을 어떻게 알까?

사용자 폴더의 .kube파일의 config파일 때문이다. 이 파일은 가상 머신에서 실행되는 minikube 클러스터에 연결할 수 있게 하는 정보를 갖고있다. 이 파일을 EKS 클러스터와 통신할 수 있게 변경할 것이다. 기존의 config파일은 config.minikube로 minikube와 통신하고 싶으면 돌아갈 수 있게 미리 복사해두자.

변경시엔 AWS CLI를 이용하자. AWS 계정에 대한 명령을 로컬 머신의 CLI에서 실행할 수 있게 해준다. AWS CLI를 기본값으로 설치한다.

AWS 계정의 My Security Credential에서 Access Key를 찾아 다운받는다. 
개발 툴 명령줄에 aws configure 입력. 액세스키와 시크릿키 입력. 클러스터 설정했던 리전 입력 ex) us-east-2 그 뒤 다시 엔터

AWS CLI는 AWS 계정과 통신하고 작업 수행할 수 있게됨. 

aws eks —region us-east-2 update-kubeconfig —name 클러스터 이름 

AWS 클러스터와 통신하는데 필요한 모든 데이터로 아까 config파일을 업데이트 한다.

이제 aws 클러스터와 통신가능함.

# 워커노드 추가하기

클러스터 섹션의 노드 그룹에서 Add Node Group 클릭. 그룹 이름 지정.

워커 노드에도 로그파일 작성 등 특정 권한이 필요하다. 이 노드 그룹에서 구동될 리모트 머신인 EC2 인스턴스에 연결해야한다. IAM 역할 지정.
 Iam 에서 EC2를 선택하고 eksworker 검색하여 워크 노드 정책추가. cni, EC2containerRegistryReadOnly 추가. 

이러한 권한은 노드, 즉 클러스터의 일부인 EC2인스턴스가 이미지를 가져와 실행하기 위해 수행하는 다양한 작업 허용. 
역할 이름 지정하고 생성.

클러스터 노드 그룹에서 추가해준다. 

여기서는 AWS에 의해 어떤 종류의 EC2인스턴스가 구동되고 관리될 것인지 제어한다. 
t3.small이 최소값이다. micro를 사용한다면 Pod 스케줄링 과정에서 Pending 상태가 될것이다. 
2개의 노드를 갖는것은 2개의 다른 인스턴스를 갖는것이다. Pod와 컨테이너는 쿠버네티스에 의해 자동으로 배포된다. 

Allow remote access to nodes 옵션을 끄면 SSH를 통해 이러한 노드, EC2인스턴스에 직접 연결할 수 없게된다. 끄고 진행.

생성한다.

이것은 몇개의 EC2 인스턴스를 가동하여 클러스터에 추가한다. EKS는 이러한 노드에 kubelet 및 kube-proxy에 필요한 쿠버네티스 소프트웨어도 설치하고 연결한다. 원래는 직접 해야함.

애플리케이션 배포전, EC2 대시보드를 보면 로드 밸런서가 표시되지 않는다. 

# k8s 구성 적용하기

- docker-compose 와 deployment,service.yaml
    
    **Kubernetes 환경에서는 `docker-compose` 파일이 필요하지 않습니다. `docker-compose`는 Docker 환경에서 여러 컨테이너를 정의하고 관리하기 위한 도구로, 주로 로컬 개발 환경이나 간단한 배포에 사용됩니다.**
    
    **반면, Kubernetes는 클러스터 환경에서 애플리케이션을 배포하고 관리하기 위한 플랫폼으로, `deployment.yaml`, `service.yaml` 등과 같은 YAML 파일을 사용하여 리소스를 정의합니다. Kubernetes는 더 복잡한 배포 시나리오를 지원하며, 자동 스케일링, 롤링 업데이트, 서비스 디스커버리 등 다양한 기능을 제공합니다.**
    
    **따라서 Kubernetes를 사용할 경우, `docker-compose` 파일 없이도 `deployment.yaml`과 같은 Kubernetes 리소스 정의 파일만으로 애플리케이션을 배포할 수 있습니다. 다만, Docker Compose를 사용하여 로컬에서 개발하고 테스트한 후, Kubernetes로 배포하는 경우도 많습니다. 이 경우에는 Docker Compose 파일이 초기 개발 단계에서 유용할 수 있습니다.**
    
    **그럼 왜 굳이 docker-compose로 다중 컨테이너 실행시키고 deployment 파일을 빌드하는거야 ?**
    
    **Docker Compose와 Kubernetes는 각각의 장점이 있으며, 서로 다른 상황에서 유용하게 사용됩니다. Docker Compose를 사용하여 다중 컨테이너를 실행하고, 이후에 Kubernetes의 Deployment 파일을 빌드하는 이유는 다음과 같습니다:**
    
    - **로컬 개발 환경: Docker Compose는 로컬 개발 환경에서 여러 컨테이너를 쉽게 설정하고 실행할 수 있도록 도와줍니다. 개발자는 복잡한 설정 없이도 필요한 서비스(예: 데이터베이스, 캐시 서버 등)를 빠르게 구성할 수 있습니다.**
    - **빠른 프로토타이핑: Docker Compose를 사용하면 애플리케이션의 여러 구성 요소를 신속하게 조합하여 테스트할 수 있습니다. 이는 개발 초기 단계에서 유용합니다.**
    - **단순한 설정: Docker Compose 파일은 YAML 형식으로 간단하게 작성할 수 있어, 여러 컨테이너의 설정을 직관적으로 관리할 수 있습니다. Kubernetes의 YAML 파일보다 상대적으로 간단합니다.**
    - **이식성: Docker Compose 파일은 다양한 환경에서 쉽게 이식할 수 있습니다. 개발자가 로컬에서 작업한 내용을 그대로 다른 환경(예: CI/CD 파이프라인)으로 옮길 수 있습니다.**
    - **Kubernetes로의 전환: Docker Compose로 개발한 후, 애플리케이션이 안정화되면 Kubernetes로 배포하는 것이 일반적입니다. 이 과정에서 Docker Compose의 설정을 바탕으로 Kubernetes의 Deployment 및 Service 파일을 작성할 수 있습니다.**
    
    **결론적으로, Docker Compose는 개발과 테스트 단계에서 유용하며, Kubernetes는 프로덕션 환경에서의 배포와 관리를 위한 강력한 도구입니다. 두 도구를 적절히 조합하여 사용하는 것이 일반적입니다.**
    

이미지 푸쉬 후 deployment 실행. 애플리케이션이 정상적으로 올라간다.

kubectl get service로 요청보내는데 사용할 url 얻어서 사용가능.  minikube service 서비스명 같은 명령어가 필요없다.

요청을 보낸 후 대시보드에 로드 밸런서를 확인하면 service.yaml에 type을 LoadBalancer로 지정했기 때문에 마찬가지로 로드 밸런서가 생성된것을 볼 수 있다. 

# 볼륨으로 시작하기

emptyDir 은 Pod마다 볼륨이 생성되고 볼륨이 Pod와 생명주기가 같아지므로 Pod가 내려가면 볼륨도 사라지게 된다. 

hostPath는 한 노드에만 생성되고 다른 노드는 생성되지 않는다. 

- Pod의 이동: Kubernetes는 Pod의 상태를 관리하고, 필요에 따라 Pod를 다른 노드로 이동하거나 재스케줄링할 수 있습니다. 예를 들어, 노드가 다운되거나 리소스가 부족할 경우, Kubernetes는 Pod를 다른 노드로 옮길 수 있습니다.
- `hostPath`의 한계: `hostPath`를 사용하면 특정 노드의 파일 시스템에 의존하게 됩니다. 만약 Pod가 이동한 새로운 노드에 동일한 경로의 디렉토리나 파일이 없다면, Pod는 정상적으로 실행되지 않을 수 있습니다. 즉, `hostPath`가 지정된 경로가 새로운 노드에 존재하지 않으면, 해당 Pod는 필요한 리소스를 찾지 못해 오류가 발생할 수 있습니다.

hostPath 볼륨은 동일한 노드에서 실행되는 Pod들 사이에서만 데이터를 공유할 수 있습니다. 이는 멀티 노드 환경에서 데이터의 일관성과 가용성을 보장하기 어렵게 만듭니다. 

따라서 단일 노드 설정인 경우에만 유용하다.

써드파티 개발자가 자체 통합 및 자체 볼륨 유형을 쉽게 추가할 수 있는 CSI가 적합하다. 

AWS EFS를 이용하여 데이터를 관리하려고 한다. 

EFS용 CSI패키지가 존재한다. EFS를 볼륨 유형, 볼륨 드라이버로 사용할 수 있는 매우 유용한 통합이다. https://github.com/kubernetes-sigs/aws-efs-csi-driver

```yaml
kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-2.0"
```

로 쿠버네티스 클러스터에 드라이버를 설치한다. 

AWS EFS가 볼륨 타입을 지원하지 않아서 드라이버가 필요하다.

EFS(탄력적 파일 시스템)을 만들어야한다. 

EC2 보안 그룹에서 vpc를 기존에 만든 그룹으로 선택한다. 그래야 쿠버네티스 클러스터용으로 생성된 네트워크의 VPC에서 보안그룹이 작동하기 때문이다.

인바운드 보안 규칙은 NFS 타입을 선택(2049번 포트) CIDR 범위는 VPC 그룹의 IPV4를 붙혀넣는다. (VPC서비스 탭에서 확인해야함)

EFS 생성하는데 VPC는 기존 그룹 선택. Customize를 선택하고 네트워크 액세스 페이지에서 가용영역에 기존 보안그룹 제거하고 위에 생성한 보안그룹으로 바꿔준다. 그리고 생성.

볼륨으로 사용가능한 파일 시스템이 만들어진다. 파일시스템 ID 복사한다. 

# EFS 파일시스템을 볼륨으로 추가해보자

[영구볼륨](https://www.notion.so/k8s-10c97d6ce88f80e5a90bfdbf7d194a28?pvs=21)에서만 작동한다. 

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: efs-pv
spec:
  capacity: 
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany #여러 노드가 같이 사용
  storageClassName: efs-sc
  csi:
    driver: efs.csi.aws.com #공식문서 예제를 찾아봐야함
    volumeHandle: fs-59d14521 # 아까 복사한 파일시스템 ID
```

EFS를 위한 특정 STorageClass 가 필요하지만 현재 쿠버네티스 클러스터에 존재하지 않으므로 이 클래스를 가져오기 위해 밑과 같은 정보 추가한다.

https://github.com/kubernetes-sigs/aws-efs-csi-driver/blob/master/examples/kubernetes/static_provisioning/specs/storageclass.yaml

```yaml
kubectl get sc # 스토리지 목록 보기 
```

****`StorageClass`는 Kubernetes에서 스토리지 리소스를 동적으로 생성하고 관리하는 데 필요한 설정을 정의하는 리소스입니다.

즉, EFS를 사용하려면 다음과 같은 이유로 `StorageClass`가 필요합니다:

- 스토리지 프로비저닝: Kubernetes에서 Persistent Volume Claim(PVC)을 통해 스토리지를 요청할 때, 어떤 종류의 스토리지를 사용할 것인지 지정해야 합니다. 이때 EFS를 사용하려면 EFS에 대한 `StorageClass`가 필요합니다.
- 동적 생성: `StorageClass`가 없으면 PVC를 요청할 때 Kubernetes가 어떤 스토리지를 생성해야 할지 알 수 없습니다. `StorageClass`를 정의함으로써, Kubernetes는 EFS를 사용하여 필요한 스토리지를 자동으로 생성할 수 있습니다.
- 설정 및 관리: `StorageClass`를 통해 EFS의 설정(예: 성능, 리소스 할당 등)을 관리할 수 있습니다. 이를 통해 사용자는 필요에 따라 다양한 스토리지 옵션을 선택할 수 있습니다.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com # 이 드라이버를 가리켜 스토리지 클래스 구성한다
```

이를 통해 StorageClass와 PV를 가지게 된다. 

이제 Pod 내부에서, 컨테이너 내부에서 특정 볼륨 요구하여 컨테이너가 이를 [볼륨으로 사용](https://www.notion.so/k8s-10c97d6ce88f80e5a90bfdbf7d194a28?pvs=21)하도록 하면 된다

```yaml
     volumeMounts:
            - name: efs-vol
              mountPath: /app/users #컨테이너 내부 경로, 여기서 /app은 Dockerfile에 작업 디렉토리 경로로 설정한 곳이다.
      volumes:
        - name: efs-vol
          persistentVolumeClaim: 
            claimName: efs-pvc

```

Pod의 클레임에 액세스 하려는 경우, 해당 클레임네임을 사용하면 된다. 볼륨 마운트 추가하여 이 볼륨을 컨테이너의 특정 경로에 마운트한다.  

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
```


![3](https://github.com/user-attachments/assets/dad75b12-1447-46f8-b10d-fc8b9ce9b20a)
![2](https://github.com/user-attachments/assets/ed2984c8-3bb4-4526-a5bf-21746aff7cc4)
![1](https://github.com/user-attachments/assets/f2978c66-de0d-4fa2-aa04-7f0937ff88a1)




