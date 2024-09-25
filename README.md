# [ Trivy : Docker Image Vulnerability 진단 ]

## 🧹 프로젝트 개요
Docker 이미지 취약점 진단 도구인 Trivy에 탐구하고, **Trivy와 GitHub Actions**를 활용해 Docker 이미지를 빌드할 때 자동으로 취약점 검사를 수행하고 스캔 결과를 GitHub의 Code scanning과 Slack에 리포트하는 워크플로우를 구성했습니다.

<br>

## 🎡 실행 환경 및 기술 스택
- Ubuntu 22.04
- Docker 27.3.1
- Docker Hub
- Slack

<br>
<h2 style="font-size: 25px;"> 👨‍👨‍👧‍👦💻 팀원 <br>
<br>
    
|<img src="https://avatars.githubusercontent.com/u/64997345?v=4" width="120" height="120"/>|<img src="https://avatars.githubusercontent.com/u/38968449?v=4" width="120" height="120"/>
|:-:|:-:|
|[@최영하](https://github.com/ChoiYoungha)|[@허예은](https://github.com/yyyeun)

</h2>
<br>

## 🔎 About Trivy
### 💡 Trivy란
<div align="center">
<img src="https://github.com/user-attachments/assets/981bb83c-7783-407f-b264-b7ca9425df2a" width="400">
</div><br>

**보안 취약점을 스캔**하기 위한 도구로, 다음과 같은 기능을 수행합니다.

- **취약점 탐지**
    - CVE(Common Vulnerabilities and Exposures): 알려진 보안 취약점을 탐지
    - SBOM(Software Bill of Materials): 사용 중인 운영체제(OS) 패키지와 소프트웨어 의존성에서 취약점을 확인

- **잘못된 설정 (Misconfigurations)**
    - IaC(Infrastructure as Code): 잘못된 인프라 설정을 검사
    - 민감한 정보와 비밀: 이미지나 리소스에서 중요한 정보나 비밀이 노출된 경우 탐지


### Trivy가 스캔 가능한 artifact
1. Container image : Docker 및 기타 컨테이너 이미지에 대한 취약점 스캔
2. File system 및 Rootfs: 파일 시스템과 루트 파일 시스템의 보안 문제 탐지
3. Git Repository: 코드베이스의 보안 문제와 잘못된 설정을 확인
4. Kubernetes: 쿠버네티스 클러스터 및 설정의 보안 문제 탐지

<br>

## 👜 실습 환경 준비
### 0. Installing Docker
Docker 설치가 선행되어야 하며, 빌드한 이미지를 업로드할 Docker Hub와 계정이 필요합니다.

<br>

### 1. Installing Trivy
[Official docs](https://aquasecurity.github.io/trivy/v0.19.2/getting-started/installation/)를 참고해 자신의 OS에 맞는 install 과정 수행
```
sudo apt-get install wget apt-transport-https gnupg lsb-release

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list

sudo apt-get update

sudo apt-get install trivy
```

<br>

### 2. Trivy Test : Running a Vulnerability Scan
```
trivy image <Image ID or Image Name>
``` 

<br><div align="center">
<img src="https://github.com/user-attachments/assets/bf1bdb9e-488e-45d7-a017-a34b6dfa830e" width="800">
</div><br>

[Docker 이미지 최적화](https://github.com/yyyeun/WooriFISA3-Docker-ImageOptimization) 프로젝트에서 [생성한 이미지](https://hub.docker.com/r/gymlet/spring_optimization/tags)의 취약점을 검사한 모습입니다.


<br> 

## 🎇 GitHub Actions와 결합 : Docker 이미지 빌드 시 취약점 검사 수행
GitHub Actions yml 파일을 생성하고, 자동화할 동작들을 정의합니다.

<div align="center">
<img src="https://github.com/user-attachments/assets/4d87a4bb-0da1-4caa-86d5-91e707752c96" width="600">
</div><br>

```
# 1. Repository 체크아웃
- uses: actions/checkout@v4

# 2. Docker 로그인
- name: Log in to Docker Hub
  run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

# 3. Docker 이미지 빌드
- name: Build Docker image
  run: docker build -t ${{ secrets.DOCKER_USERNAME }}/my-trivy-image:latest .
    
# 4. Trivy 설치
- name: Install Trivy
  run: |
    sudo apt-get update
    sudo apt-get install -y wget apt-transport-https gnupg lsb-release
    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
    echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
    sudo apt-get update
    sudo apt-get install trivy
    
# 5. Docker 이미지 취약점 스캔
- name: Scan Docker image for vulnerabilities
  run: trivy image --format sarif --output trivy-results.sarif ${{ secrets.DOCKER_USERNAME }}/my-trivy-image:latest
    
# 6. Docker Hub에 이미지 Push
- name: Push Docker image to Docker Hub
  if: success()
  run: docker push ${{ secrets.DOCKER_USERNAME }}/my-trivy-image:latest
    
# 7. 취약점 검사 결과를 GitHub Code Scanning으로 업로드
- name: Upload Trivy scan results to GitHub
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: trivy-results.sarif
```

<br>

> 참고: yml 파일에서 변수를 사용하기 위해서는 GitHub Secrets 설정이 필요합니다.

<br>

## 🎨 최종 실행 결과

<div align="center">
<img src="https://github.com/user-attachments/assets/e22b10d4-a30a-4bac-b200-3b4142cd8723" width="700">
</div><br>

<div align="center">
<img src="https://github.com/user-attachments/assets/9ccfdedd-c7ae-4ef9-821f-90efdc82adc0" width="700">
</div><br>

`main` 브랜치로 push 또는 Pull Request 동작이 수행되면 GitHub Actions가 수행됩니다.<br>
정상적으로 완료되면 Code scanning 탭에 진단된 취약점이 반영되며, Slack으로 Critical과 High에 해당하는 취약점이 리포트됩니다.

<br>

## 🧵 결론 및 고찰
> Trivy를 사용해 앞서 최적화를 진행한 Docker 이미지에 대한 취약점을 직접 진단함으로써 보안 강화의 중요성을 재고했으며, GitHub Actions와 Code scanning 등 GitHub에서 제공하는 유용한 기능들을 활용해볼 수 있었습니다.
