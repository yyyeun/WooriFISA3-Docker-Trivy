# [ Trivy : 도커 이미지 취약점 진단 ]

## 🧹 프로젝트 개요
> Trivy 탐구 및 실습

<br>
<h2 style="font-size: 25px;"> 👨‍👨‍👧‍👦💻 팀원 <br>
<br>
    
|<img src="https://avatars.githubusercontent.com/u/64997345?v=4" width="120" height="120"/>|<img src="https://avatars.githubusercontent.com/u/38968449?v=4" width="120" height="120"/>
|:-:|:-:|
|[@최영하](https://github.com/ChoiYoungha)|[@허예은](https://github.com/yyyeun)

</h2>
<br>

## 🕶 About Trivy
### Trivy란
<div align="center">
<img src="https://github.com/user-attachments/assets/981bb83c-7783-407f-b264-b7ca9425df2a" width="400">
</div><br>
보안 취약점을 스캔하기 위한 도구로, Docker Hub와 같은 곳에서 이미지의 취약점을 찾는 데 사용할 수 있습니다.
- 취약점 탐지
    - CVE(Common Vulnerabilities and Exposures): 알려진 보안 취약점을 탐지합니다.
    - SBOM(Software Bill of Materials): 사용 중인 운영체제(OS) 패키지와 소프트웨어 의존성에서 취약점을 확인합니다.

- 잘못된 설정 (Misconfigurations)
    - IaC(Infrastructure as Code): 잘못된 인프라 설정을 찾아냅니다.
    - 민감한 정보와 비밀: 이미지나 리소스에서 중요한 정보나 비밀이 노출된 경우 탐지합니다.

### Trivy가 스캔 가능한 artifact
1. 컨테이너 이미지: Docker 및 기타 컨테이너 이미지에 대한 취약점 스캔
2. 파일시스템 및 Rootfs: 파일 시스템과 루트 파일 시스템의 보안 문제 탐지
3. Git 리포지토리: 코드베이스의 보안 문제와 잘못된 설정을 확인
4. Kubernetes: 쿠버네티스 클러스터 및 설정의 보안 문제 탐지

<br>

## 👜 실습 환경 준비
### 1. Installing Trivy
[Official docs](https://aquasecurity.github.io/trivy/v0.19.2/getting-started/installation/)를 참고해 자신의 OS에 맞는 install 과정 수행
```
sudo apt-get install wget apt-transport-https gnupg lsb-release

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list

sudo apt-get update

sudo apt-get install trivy
```

### 2. Trivy Test : Running a Vulnerability Scan
```
trivy image <Image ID or Image Name>
``` 
<br><div align="center">
<img src="https://github.com/user-attachments/assets/bf1bdb9e-488e-45d7-a017-a34b6dfa830e" width="460">
</div><br>

[도커 이미지 최적화](https://github.com/yyyeun/WooriFISA3-Docker-ImageOptimization) 프로젝트에서 생성한 [이미지](https://hub.docker.com/r/gymlet/spring_optimization/tags)의 취약점을 검사한 모습입니다.


<br> 

## 🎇 Trivy 활용 예제 : 

<br>

## 🎨 최종 실행 결과

<br>

## 🧵 결론 및 고찰
> 