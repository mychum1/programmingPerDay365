## 쿠버네티스 설치 방법
### 직접 설치 
1. k3s : IoT 디바이스에서 동작할 수 있을 정도로 경량화 되어있다. 설치가 쉽고 무료..!
2. minikube
3. microk8s : ubuntu 에서 제공
4. kubeadm : 공식 제공

### 완전관리형 서비스
1. EKS - AWS
2. GKE - GCP
3. AKS - AZURE

#### k3s 설치
1. sudo apt-get update //apt 업데이트부터 시작
2. sudo apt-get install -y docker.io ngs-common python3-dev python3-pip //설치할 것들
3. curl -sfL https://get.k3s.io | INSTALL_K#S_EXEC=“\
 --disable traefik \
 --disable metrics-server \
 --node-name master - -docker” \
INSTALL_K3S_VERSION=“v1.17.4+k3s1” sh -s - 
4. mkdir ~/ .kube
5. sudo cp -etc/rancher/k3s/k3s.yaml ~/ .kube/config
6. sudo chown -R $(id -u):$(id -g) ~/ .kube
7. source ~/ .bashrc
8. kubectl get node

1~2 : 설치 전 해야할 것들
3: 설치
4~7: sudo를 매번 치지 않아도 되게끔 설정
8: 테스트

## kubectl 명령어

1. Pod 실행하기
	A. kubectl run {컨테이너 이름} -- image {이미지 이름} --restart Never 
2. Pod 조회하기
	A. kubectl get pod {컨테이너 이름}
3. Yaml 정의서 확인하기 ( o: output. yaml 과 붙여써도 됨)
	A. kubectl get pod {컨테이너 이름} -o yaml
	B. kubectl get pod {컨테이너 이름} -oyaml
4. Pod 상태 확인
	A. kubectl describe pod {컨테이너 이름}
5. Pod 명령 전달하기
	A. kubectl exec {컨테이너 이름} -- apt update //apt update 명령어를 전달
6. 로그 확인
	A. kubectl logs {컨테이너 이름}
7. 로컬호스트에 있는 파일을 pod 로 복사
	A. kubectl cp ~/ .bashrc {컨테이너 이름}:/.
8. Pod 수정
	A. kubectl edit pod {컨테이너 이름} /앱 정의서 yaml 을 수정
9. Pod 삭제
	A. kubectl delete pod {컨테이너 이름}
10. 네임스페이스 // 리소스 crud. 네임스페이스를 설정하지 않으면 default 네임스페이스에서 crud가 됨
	A. kubectl get pod --namespace kube-system //kube-system에 있는 네임스페이스들을 리스팅한다.
	B. kubectl get pod -n kube-system //--namespace = -n
11. 라벨 부여하기
	A. kubectl label node {node 이름} {라벨 이름}
		ex)kubectl label node master color-red
		확인 ) kubectl get node master -o yaml
12. 모든 리소스 조회
	A. kubectl api-resources
13. 모든 서비스 조회
	A. kubectl services
14. 앱정의서의 내용 조회
	A. kubectl get pod {pod name} -o jsonpath=“{~~~}”
		jsonpath ex. .spec.containers[0].image
		// spec의 0번째 컨테이너의 이미지 출력
15. 자동완성기능 사용
	A. echo “source <(kubectl completion bash)” >> ~/.bashrc
	B. source ~/.bashrc
16. 사용자 인증파일 조회
	A. kubectl config view
17. 클러스터 정보 조회
	A. kubectl cluster-info

## Pod 에 대하여
쿠버네티스의 최소 단위이다.
	1. 한 개 이상의 컨테이너를 실행한다.
	2. IP를 공유한다. node ip != pod ip, pod ip = container ip
	3. 동일 노드에 할당된다.
![파드 이미지](https://github.com/mychum1/programmingPerDay365/blob/master/images/AF496B42-3CA1-4C2F-A942-A24100AC751A.jpeg)

## 앱정의서(선언형 명령어)에 대하여
1. apiVersion : scope. (자바의 패키지같은 존재)
2. kind : 리소스 타입(자바의 클래스같은 존재)
3. metadata : 메타정보
	3_1 labels : key:value 로 이루어진 라벨 정보
	3_2 name : 리소스의 이름.
4. spec : 리소스 정의
	4_1 containers : 컨테이너 정의 
		4_1_1 name: 컨테이너 이름
		4_1_2 image : 컨테이너 이미지 
로 작성 후,
kubectl apply -f 앱정의서yaml

## 즉석해서 리소스를 생성하기
cat << EOF | kubectl apply -f -
를 사용하면 car redirection을 이용해서 즉석으로 yaml 파일을 만들어줌. (포맷)

## 라벨링 시스템
### Pod 에 라벨링하기.
1. kubectl get pod -L {label key} //어떤 라벨의 pod가 동작하고 있는지 확인
2. kubectl get pod {pod name} —show-labels //pod 들의 라벨들을 확인
3. kubectl get pod -l {label key}
4. kubectl get pod -l {label key}={label value}

### nodeSelector 로 셀렉하기
앱정의서에 nodeSelector 설정 //특정 라벨의 노드에 반영되길 원할 때 사용

## Volume 리소스 연결
1. host local volume 을 연결하기 
1.1 앱 정의서에 volumes, volumesMounts 추가하기
* volumes : 로컬의 디렉토리 위치 지정
* volumesMounts : 컨테이너 내부의 디렉토리 위치 지정
1.2 kubectl apply -f {앱 정의서}
