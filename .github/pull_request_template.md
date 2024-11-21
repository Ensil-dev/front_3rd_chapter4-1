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

<br/>


# CDN 도입 전과 도입 후의 성능 개선 Report [심화 과제]

## 1️⃣ 개요

S3 단독 배포 방식은 글로벌 사용자에게 리소스를 전달하는 데 한계가 있을 수 있습니다. 이를 확인하고 해결하기 위해 CloudFront를 도입하여 S3와 연계한 배포 방식을 적용했습니다. 이 과정에서 초기 응답 시간, 리소스 로딩 속도, 파일 크기 최적화 등에서 성능 개선 여부를 구체적으로 측정했습니다. 분석 결과, S3와 CDN 연계를 통해 기술적 이점뿐 아니라 사용자 경험의 체감 로딩 속도도 크게 향상되었습니다.

<br/>


## 2️⃣ 주요 파일별 로딩 시간 개선

### 2-1) CDN 도입 전후 네트워크 요청 비교

CDN 도입 전 네트워크 요청           |                | CDN 도입 후 네트워크 요청
:----------------------------------:|:--------------:|:----------------------------------:
![CDN 도입 전](https://raw.githubusercontent.com/Ensil-dev/front_3rd_chapter4-1/refs/heads/chapter4-1-jungyoon/public/cdn-before-network-request.png)          |                | ![CDN 도입 후](https://raw.githubusercontent.com/Ensil-dev/front_3rd_chapter4-1/refs/heads/chapter4-1-jungyoon/public/cdn-after-network-request.png)


### 2-2) 주요 파일별 로딩 시간 개선
| 파일 유형   | CDN 전 | CDN 후 | 개선율    |
|-------------|--------|--------|-----------|
| HTML 문서   | 97ms   | 10ms   | 89.7%   |
| 스타일시트   | 53ms   | 20ms   | 62.3%   |
| 스크립트    | 154ms  | 34ms   | 77.9%   |
| 폰트        | 115ms  | 31ms   | 73.0%   |
| SVG         | 54ms   | 26ms   | 51.9%   |

### 2-3) 파일 크기 최적화

| 항목     | CDN 전   | CDN 후   | 개선율    |
|----------|----------|----------|-----------|
| HTML     | 12.5 kB  | 3.2 kB   | 74.4%  |
| CSS      | 9.0 kB   | 2.9 kB   | 67.8%  |
| 메인 JS  | 166 kB   | 49.7 kB  | 70.1%  |

<br/>


## 3️⃣ 주요 성능 지표

### 3-1) CDN 도입 전후 성능 통계 비교

CDN 도입 전 성능 통계           |                | CDN 도입 후 성능 통계
:----------------------------------:|:--------------:|:----------------------------------:
![CDN 도입 전](https://raw.githubusercontent.com/Ensil-dev/front_3rd_chapter4-1/refs/heads/chapter4-1-jungyoon/public/cdn-before-performance-stats.png)          |                | ![CDN 도입 후](https://raw.githubusercontent.com/Ensil-dev/front_3rd_chapter4-1/refs/heads/chapter4-1-jungyoon/public/cdn-after-performance-stats.png)


### 3-2) 주요 성능 지표 비교

| 지표                     | CDN 전  | CDN 후  | 개선율    |
|--------------------------|---------|---------|-----------|
| DOM Content Loaded       | 0.13초 | 0.03초 | 76.9%  |
| First Contentful Paint   | 0.15초 | 0.04초 | 73.3%  |
| Largest Contentful Paint | 0.15초 | 0.04초 | 73.3%  |

<br/>


## 4️⃣ 분석

### 4-1) CDN 캐싱 효과
- Edge Location 활용으로 초기 응답 시간이 **89.7% 감소**했습니다.
- 정적 리소스 전송 시간을 평균 **70% 이상 단축**했습니다.
- CDN 캐싱은 글로벌 사용자들에게 **안정적이고 빠른 응답 시간**을 제공합니다.
- CDN은 원본 서버로의 요청을 줄여 **서버 부하를 감소**시키는 효과가 있습니다.

### 4-2) 성능 지표 개선
- FCP/LCP에서 **70% 이상의 렌더링 성능 향상**이 있었습니다.
- DOM 로딩 시간 **76.9% 단축으로 페이지 응답성이 크게 개선** 되었습니다.
- 전체 페이지 로드에 필요한 **리소스 크기가 70% 이상 감소** 했습니다.

### 4-3) 사용자 경험 향상
- 체감 로딩 속도가 **70% 이상 개선** 되었습니다.
- HTTPS 적용으로 **보안성이 강화** 되었습니다.
- **글로벌 사용자 접근성 향상** 되었습니다.

<br/>


## 5️⃣ 결론

CloudFront를 적용함으로써 S3 단독 호스팅 대비 전반적인 성능이 70% 이상 개선되었습니다. 또한 Edge Location을 활용해 전 세계 사용자에게 지연 시간을 최소화하고 높은 성능을 제공할 수 있게 되었으며, HTTPS 적용을 통해 보안 취약점을 해소했습니다.