# 프론트엔드 배포 파이프라인 Report [기본 과제]

## 배포 파이프라인 Diagram
![배포 CI/CD Diagram](https://raw.githubusercontent.com/Ensil-dev/front_3rd_chapter4-1/refs/heads/chapter4-1-jungyoon/public/diagram.webp)

## 주요 단계 설명

### 1️⃣ Git Repository
+ 작업 branch(chapter4-1-jungyoon)에서 push 시 자동으로 PR을 생성합니다.
![image](https://github.com/user-attachments/assets/10252d65-1643-4495-b3f3-fd33f7164520)

+ 메인테이너에 의해 병합이 승인되어 main 브랜치에 push 되면 CD 파이프라인이 시작됩니다.
![image](https://github.com/user-attachments/assets/7bf00e9c-152b-49b0-b871-f82e9a86c49b)

### 2️⃣ GitHub Actions

1) **저장소 체크아웃**
+ GitHub Actions에서 코드를 가져오는 체크아웃 단계를 표시합니다.
  * `actions/checkout@v4`를 사용해 GitHub Actions에서 현재 저장소의 코드를 가져옵니다. 이를 통해 빌드와 배포에 필요한 코드베이스를 워크플로우가 사용할 수 있도록 합니다.

2) **Node.js 설정**
+ Node.js 20 버전을 설치하고 설정합니다.
  * `actions/setup-node@v4`를 사용하여 Node.js 20.x 버전을 설치하고, `npm` 캐시를 사용해 의존성 설치 시간을 줄입니다. 이렇게 함으로써 필요한 Node.js 환경을 설정하고, 빌드와 패키지 설치가 원활히 이루어질 수 있습니다.

3) **의존성 설치**
+ npm ci 명령을 통해 프로젝트 의존성을 설치하는 작업을 표시합니다.
  * `npm ci` 명령어를 사용해 의존성을 설치합니다. 이는 `package-lock.json` 파일을 기준으로 의존성을 엄격하게 설치하여 빌드 과정의 일관성을 보장합니다.

4) **Next.js 프로젝트 빌드**
+ Next.js 프로젝트를 빌드하는 작업을 추가합니다.
  * `npm run build` 명령어를 통해 Next.js 프로젝트를 빌드합니다. 빌드된 결과물은 `out` 디렉토리에 저장되며, 이후 S3에 업로드됩니다.

5) **AWS 자격 증명 구성**
+ GitHub Actions가 AWS S3 및 CloudFront에 접근할 수 있도록 자격 증명을 설정합니다.
  * GitHub Secrets에 저장된 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` 정보를 사용하여 AWS 리소스에 접근할 수 있게 합니다.

### 3️⃣ Amazon S3
+ 빌드 산출물을 AWS S3에 업로드하는 단계입니다.
  * `aws s3 sync` 명령어를 사용해 빌드된 `out` 디렉토리의 파일들을 S3 버킷으로 동기화합니다.
  * `--delete` 옵션으로 S3 버킷에 있는 기존 파일 중 불필요한 파일을 삭제하여 최신 버전의 파일만 유지되도록 합니다.

### 4️⃣ Amazon Cloudfront
+ CloudFront 캐시 무효화가 실행됩니다.
  * `aws cloudfront create-invalidation` 명령어를 통해 CloudFront의 기존 캐시를 무효화합니다.
  * 사용자가 항상 최신 버전의 콘텐츠를 받을 수 있도록 합니다.

### 5️⃣ Internet
+ 최종적으로 사용자들이 Amazon CloudFront를 통해 콘텐츠에 접근합니다.

<br/>

## 주요 링크

+ **S3 버킷 웹사이트 엔드포인트**: [`http://hanhaeplus-fe-chapter-4-1.s3-website.ap-northeast-2.amazonaws.com`](http://hanhaeplus-fe-chapter-4-1.s3-website.ap-northeast-2.amazonaws.com)

+ **CloudFront 배포 도메인 이름**: [`d3ko9vd7loegqu.cloudfront.net`](http://d3ko9vd7loegqu.cloudfront.net)

<br/>

## 주요 개념

### GitHub Actions과 CI/CD 도구
+ **GitHub Actions**는 GitHub에서 제공하는 CI/CD 도구로, 코드 저장소에서 특정 이벤트(예: push, pull request)가 발생했을 때 자동으로 빌드, 테스트, 배포 등의 작업을 수행할 수 있습니다.  
  * ``CI (Continuous Integration)``는 **코드 변경 시 자동으로 테스트와 빌드를 수행**하여 변경 사항이 잘 통합되었는지 확인하는 프로세스입니다.
  * ``CD (Continuous Deployment/Delivery)``는 테스트 및 빌드 완료 후 **변경 사항을 자동 또는 수동으로 배포하는 과정**을 의미합니다.

### S3와 스토리지
+ ``Amazon S3 (Simple Storage Service)``는 AWS에서 제공하는 객체 스토리지 서비스로, **데이터를 안전하게 저장하고 웹사이트의 정적 파일을 호스팅하는 데 사용**됩니다. 
+ 이 프로젝트에서는 Next.js 빌드 결과를 S3 버킷에 업로드하여 사용자에게 정적 파일로 제공합니다.

### CloudFront와 CDN
+ ``Amazon CloudFront``는 AWS의 콘텐츠 전송 네트워크(CDN) 서비스입니다. **전 세계의 엣지 로케이션을 활용하여 사용자에게 더 빠르게 콘텐츠를 제공**할 수 있습니다. 
+ 이 배포 파이프라인에서는 S3의 정적 파일을 CloudFront를 통해 사용자에게 전송하여 성능과 안정성을 높입니다.

### 캐시 무효화 (Cache Invalidation)
+ ``CloudFront 캐시 무효화``는 **CloudFront 엣지 로케이션에 캐시된 파일이 최신 상태가 아닐 때 이를 무효화하여, S3의 최신 파일을 제공하도록 하는 작업**입니다. 캐시 무효화를 통해 웹사이트에 적용된 변경 사항이 사용자의 브라우저에 빠르게 반영될 수 있습니다.

### Repository secret과 환경변수
+ ``GitHub Secrets``는 **민감한 정보를 안전하게 저장하고 관리하는 기능**으로, AWS 자격 증명이나 API 키와 같은 정보를 포함합니다. 이 정보들은 GitHub Actions 워크플로우에서 참조되어 배포 과정에서 안전하게 사용됩니다.
+ ``환경변수``는 코드 내에 하드코딩하지 않고 외부 설정 파일이나 환경변수로 관리하여 **보안과 유연성**을 높이는 방식입니다.
