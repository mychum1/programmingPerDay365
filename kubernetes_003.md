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
