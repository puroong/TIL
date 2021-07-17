# Serverless란

* 서버리스 어플리케이션을 쉽게 개발/배포할 수 있게 해주는 프레임워크
* serverless로 서버리스 어플리케이션 모니터링/디버깅/배포을 할 수 있다

## Installation

MacOS / Linux
```
curl -o- -L https://slss.io/install | bash
```

* 특정 버전을 설치하고 싶은 경우
```
curl -o- -L https://slss.io/install | VERSION=2.21.1 bash
```

Windows

npm으로 설치
```
npm install -g serverless
```

## Initial Setup

```
serverless
```

## Provider

* 서버리스는 AWS, Azure와 같은 cloud provider에 서버리스 어플리케이션 배포를 간단하게 해주는 프레임워크다
* 따라서 서버리스 어플리케이션을 배포하기 전에 어떤 cloud provider에 배포할지 설정해야 한다