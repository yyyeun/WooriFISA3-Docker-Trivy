# [ ✈DevSecOps : Trivy, Git Action을 이용한 Docker image 취약점 분석 알림 ]

## 🧹 개요
> 취약점 분석 자동화를 목표로합니다. 개발자들은 평소대로 git push를 사용하면 되고, 인프라 관리자는 trivy cli 명령어를 일일이 실행하지 않아도 됩니다. 코드가 push 되면 자동으로 도커 이미지를 빌드하면서 취약점을 분석하고 문제 시 슬랙으로 알람을 보냅니다.


<br>
<h2 style="font-size: 25px;"> 👨‍👨‍👧‍👦💻 팀원 <br>
<br>
    
|<img src="https://avatars.githubusercontent.com/u/64997345?v=4" width="120" height="120"/>|<img src="https://avatars.githubusercontent.com/u/38968449?v=4" width="120" height="120"/>
|:-:|:-:|
|[@최영하](https://github.com/ChoiYoungha)|[@허예은](https://github.com/yyyeun)

</h2>
<br>

## 🚍 작업 Workflow
**1. 개발자가 코드를 push하면 git action이 실행됩니다.**<br>
**2. 도커 이미지가 빌드되고 trivy로 취약점을 분석합니다.**<br>
**3. 취약점 발견 시 슬랙으로 알람이 오고, 상세 취약점은 코드 스캐너에서 확인할 수 있습니다.**
<br>

## 🕶 Trivy 선택한 이유
### Why Trivy?
<div align="center">
<img src="https://github.com/user-attachments/assets/981bb83c-7783-407f-b264-b7ca9425df2a" width="400">
</div><br>
Trivy는 단일 바이너리 파일로 제공되어 설치가 매우 간단합니다. 복잡한 설정이나 종속성이 없어 빠르게 시작할 수 있습니다.
 Trivy는 경량화된 설계로 스캔속도가 빠릅니다. 대규모 컨테이너 이미지나 프로젝트에서도 효율적으로 작동하기 때문에 선택하였습니다.
 <br>
 <br>

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

## 👜 Trivy 설치 및 테스트
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

[🐳도커 이미지 최적화](https://github.com/yyyeun/WooriFISA3-Docker-ImageOptimization) 프로젝트에서 생성한 [이미지](https://hub.docker.com/r/gymlet/spring_optimization/tags)의 취약점을 검사한 모습입니다.


<br> 

## 🍒 Trivy 설치 및 취약점 검사 실행
```yaml
- name: Install Trivy
        run: |
          sudo apt-get update
          sudo apt-get install -y wget apt-transport-https gnupg lsb-release
          wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
          echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
          sudo apt-get update
          sudo apt-get install trivy
          
      - name: Scan Docker image for vulnerabilities (JSON)
        run: trivy image --format json --severity HIGH,CRITICAL -o trivy-results.json ${{ secrets.DOCKER_USERNAME }}/my-trivy-image:latest
```

<br>

## 🍀 취약점 레벨 추출 및 슬랙전송
```yaml
- name: Check for critical/high vulnerabilities
        id: scan
        run: |
          critical=$(jq '[.Results[] | select(.Vulnerabilities != null) | .Vulnerabilities[] | select(.Severity == "CRITICAL")] | length' trivy-results.json)
          high=$(jq '[.Results[] | select(.Vulnerabilities != null) | .Vulnerabilities[] | select(.Severity == "HIGH")] | length' trivy-results.json)
          echo "Critical vulnerabilities: $critical"
          echo "High vulnerabilities: $high"
          echo "::set-output name=critical_count::$critical"
          echo "::set-output name=high_count::$high"

      - name: Send Slack notification
        if: steps.scan.outputs.critical_count != '0' || steps.scan.outputs.high_count != '0'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
        run: |
          curl -X POST -H 'Content-type: application/json' --data \
          '{"text":"도커 이미지 파일 취약점 - 🚨Critical: '${{ steps.scan.outputs.critical_count }}', ♨High: '${{ steps.scan.outputs.high_count }}'"}' \
          $SLACK_WEBHOOK_URL
```


<br>

## 🎇 최종 Git Action Workflow

```yaml
name: Docker Image Build and Scan

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write 
      security-events: write
      
    steps:
      - uses: actions/checkout@v4

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Build Docker image
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/my-trivy-image:latest .
        
      - name: Install Trivy
        run: |
          sudo apt-get update
          sudo apt-get install -y wget apt-transport-https gnupg lsb-release
          wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
          echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
          sudo apt-get update
          sudo apt-get install trivy
          
      - name: Scan Docker image for vulnerabilities(sarif)
        run: trivy image --format sarif --output trivy-results.sarif ${{ secrets.DOCKER_USERNAME }}/my-trivy-image:latest

      - name: Scan Docker image for vulnerabilities (JSON)
        run: trivy image --format json --severity HIGH,CRITICAL -o trivy-results.json ${{ secrets.DOCKER_USERNAME }}/my-trivy-image:latest

      - name: Print json file
        run: cat trivy-results.json

      - name: Check for critical/high vulnerabilities
        id: scan
        run: |
          critical=$(jq '[.Results[] | select(.Vulnerabilities != null) | .Vulnerabilities[] | select(.Severity == "CRITICAL")] | length' trivy-results.json)
          high=$(jq '[.Results[] | select(.Vulnerabilities != null) | .Vulnerabilities[] | select(.Severity == "HIGH")] | length' trivy-results.json)
          echo "Critical vulnerabilities: $critical"
          echo "High vulnerabilities: $high"
          echo "::set-output name=critical_count::$critical"
          echo "::set-output name=high_count::$high"

      - name: Send Slack notification
        if: steps.scan.outputs.critical_count != '0' || steps.scan.outputs.high_count != '0'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
        run: |
          curl -X POST -H 'Content-type: application/json' --data \
          '{"text":"도커 이미지 파일 취약점 - 🚨Critical: '${{ steps.scan.outputs.critical_count }}', ♨High: '${{ steps.scan.outputs.high_count }}'"}' \
          $SLACK_WEBHOOK_URL
        
      - name: Push Docker image to Docker Hub
        if: success()
        run: docker push ${{ secrets.DOCKER_USERNAME }}/my-trivy-image:latest
          
      - name: Upload Trivy scan results to GitHub
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: trivy-results.sarif
```
<br>

## 🍇 Workflow 실행
![2024-09-26 10 22 55](https://github.com/user-attachments/assets/304b7382-0838-44f4-b8c9-b1f43dfd3be3)

<br>

## 🎨 최종 실행 결과

**1.취약점 발견 시 슬랙 알람 전송**
![2024-09-26 10 33 35](https://github.com/user-attachments/assets/551956e5-f992-4193-8d5b-1bef8a01c79a)

<br>

**2. Code Scanning에서 취약점 상세 확인가능**
![2024-09-26 10 34 24](https://github.com/user-attachments/assets/225a5c59-8303-45a7-9057-1e3fa12cd478)


<br>

## 🧵 결론 및 고찰
> Trivy를 사용해 앞서 최적화를 진행한 Docker 이미지에 대한 취약점을 직접 진단함으로써 **보안 강화의 중요성**을 재고했으며, GitHub Actions와 Code scanning 등과 같은 기능을 활용하여 **CI 프로세스를 최적화**할 수 있었습니다.
