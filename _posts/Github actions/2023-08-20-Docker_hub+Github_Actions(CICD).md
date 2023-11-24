---
title: Docker hub + Github actions(CI/CD)
author: jimin
date: 2023-08-20 00:00:00 +0900
categories: [Github_Actions]
tags: [Github_Actions, Spring_boot, CI/CD]
pin: false
---

# Github Actions

# CI/CD

### **CI**

- `지속적인 통합(Continuous Integration)`을 의미
- 소스/버전 관리에 대한 변경 사항을 정기적으로 확인하여 모든 사람에게 동일 작업 기반을 제공
- `빌드`와 `테스트`를 자동으로 실행하여 동작을 확인하고 변경으로 인해 문제가 생기는 부분이 없도록 보장

### CD

- `지속적인 배포(Continuous Deployment)`를 의미
- CI가 성공적으로 통과하면 수동 개입 없이 해당 변경 사항이 프로덕션에 자동으로 배포

# Github Actions란?

- 지속적인 통합 및 지속적인 배포(CI/CD) 플랫폼
- `빌드`, `테스트`, `배포` 파이프라인을 자동화할 수 있음
- CI/CD 플랫폼
  - Jenkins
    - 무료
    - Reference 및 사용자층과 정보가 많음
    - Windows, macOS 또는 openSUSE, Red Hat, Ubuntu와 같은 다양한 리눅스 OS에 사용가능
    - 설치 및 사용이 간단
      <br>
  - TravisCI
    - 무료 버전과 기업용 버전(유료)이 있음
      - 오픈 소스 프로젝트 일 경우(public repo) 무료로 사용 가능
    - 간결하고 직관적인 웹 인터페이스를 제공하는 인터넷 기반의 CI 서비스
    - Github와 연동하여 Commit/Push를 기반으로 CI가 자동 동작하며, Push 외에도 Pull Request에도 반응하도록 설계되어 있음
      <br>
  - CircleCI
    - Linux, macOS, Android, and Windows 운영체제에서 사용 가능
    - 일정 한도 내에서 무료로 사용 가능
    - Git에 Push를 하면 자동으로 테스트를 진행하여 검사 결과를 알려 주어 문제점에 대해 바로 알림을 보내줌. 이를 통해 협업시 발생할 수 있는 파일충돌이나 에러를 잡을 수 있음. 그리고 문제가 없다면 서버에 배포까지 자동으로 이루어짐
      <br>
  - TeamCity
    - Linux, macOS, Windows 등의 모든 운영체제에서 사용 가능
    - DevOps 파이프라인을 세부적으로 시각화해줌
    - 테스트 기록을 유지해주고 불안정한 테스트의 경우 신뢰도 낮음으로 표시함
    - 실시간으로 모든 사항이 보고되므로 이슈를 프로젝트 팀원에게 할당가능
    - IDE (Jetbrains)에 TeamCity를 통합하면 따로 브랜치를 생성하거나 코드를 커밋할 필요 없이 빌드, 확인, 자동화 테스트 수행까지 완료할 수 있음
      <br>
  - Github Actions
    - 무료
    - Github Marketplace
    - GitHub 플랫폼 하나에서 모든 CI/CD 과정을 해결 가능

# Github Actions 이해

![전체](/assets/img/postpic/GithubActions/전체.png)

## Workflows

- 하나 이상의 Job을 실행하는 자동화 프로세스
- `YAML` 파일로 정의
- .github/workflows 에 작성
- [https://docs.github.com/ko/actions/learn-github-actions/understanding-github-actions](https://docs.github.com/ko/actions/learn-github-actions/understanding-github-actions)

## Events

- Workflow를 실행시키는 `트리거` 역할
- Github를 통해 발생하는 활동, 스캐줄링, 수동
- 다양한 방법으로 실행시킬 수 있기 때문에 CI/CD뿐만 아니라 다양한 곳에 사용 가능
- [https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#on](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#on)

## Runners

- `독립된` **가상 머신** 또는 **컨테이너**로 Job을 실행하는 서버
- 각 러너는 한 번에 하나의 Job만 실행 가능
- Ubuntu Linux, Windows, macOS 제공

## Jobs

- 일련의 Step으로 구성된 하나의 처리 단위
- 병렬로 실행되기 때문에 서로 의존성 x, 설정을 통해 의존성 주입 가능

## Steps

- 동일한 Runner 내에서 실행
- 작성한 순서대로 실행되어 서로 종속적
- 셸 스크립트, action 실행

## Actions

- 복잡하고 반복적인 작업을 재사용 할 수 있도록 제공되는 일종의 작업 공유 메커니즘
- Github Marketplace에서 수많은 action 사용 가능
- [https://github.com/marketplace?type=actions](https://github.com/marketplace?type=actions)

# Github Actions 환경변수

- 민감한 정보를 담은 환경변수는 .gitignore에 추가해 Github에 올리지 않음
- workflow 진행 시 Github 소스코드를 사용하게 되는데 환경변수가 없어 정상적으로 실행 할 수 없음

### 환경변수 등록

1. 환경변수를 등록하고 싶은 repository에 접속
2. Settings>Security>Secrets and variables>Actions>New repository secret

   ![환경변수1](/assets/img/postpic/GithubActions/환경변수1.png)

3. Name에는 변수명을, Secret에는 값을 입력, Add secret

   ![환경변수2](/assets/img/postpic/GithubActions/환경변수2.png)

4. 생성한 secret 확인, 생성한 secret의 값은 다시 확인 할 수 없음

   ![환경변수3](/assets/img/postpic/GithubActions/환경변수3.png)

- workflow 내에서 접근 시 `${{secrets.SECRET_KEY_NAME}}`으로 접근

```yaml
steps:
  - name: write env
    run: echo "MY_ENV=${{secrets.MY_ENVIRONMENT}}" >> .env
```

# 구현

![CI/CD](/assets/img/postpic/GithubActions/CI:CD%20workflow.png)

### CI

- push 또는 Pull Request시 해당 branch에서 테스트와 빌드를 진행
- Jacoco를 사용하여 테스트, 목표한 커버리지에 도달하지 못하면 merge 할 수 없음

```yaml
name: "CI"

on:
  push:
    branches:
      - main
      - develop

  pull_request:
    branches:
      - main
      - develop

env:
  DOCKER_ID: ${{ secrets.DOCKER_ID }}
  DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
  DOCKER_IMAGE_NAME: ${{ secrets.DOCKER_IMAGE_NAME }}

permissions: write-all

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: private 레포지토리 가져오기
        uses: actions/checkout@v3
        with:
          token: ${{ secrets.SUBMODULE_TOKEN }}
          submodules: recursive

      - name: JDK 11을 설치
        uses: actions/setup-java@v3
        with:
          java-version: "11"
          distribution: "temurin"

      - name: Gradle 명령 실행을 위한 권한을 부여
        run: chmod +x gradlew

      - name: Gradle build 수행
        run: ./gradlew build -P DOCKER_ID=${{ secrets.DOCKER_ID }} -P DOCKER_PASSWORD=${{ secrets.DOCKER_PASSWORD }} -P DOCKER_IMAGE_NAME=${{ secrets.DOCKER_IMAGE_NAME }}

      - name: 테스트 결과
        uses: EnricoMi/publish-unit-test-result-action@v1
        if: always()
        with:
          files: "**/build/test-results/test/TEST-*.xml"

      - name: JaCoCo Coverage 결과
        id: jacoco
        uses: madrapps/jacoco-report@v1.2
        with:
          title: 📝 테스트 커버리지 리포트입니다
          paths: ${{ github.workspace }}/build/jacoco/index.xml
          token: ${{ secrets.GITHUB_TOKEN }}
          min-coverage-overall: 80
          min-coverage-changed-files: 70
```

### CD

- CI를 통과한 branch를 merge하면 CD 실행
- CD 가상환경 내에서 도커 이미지를 생성, 이를 hub에 push
- ssh를 통해 EC2에 접속, 이미지를 pull 하고 갱신된 이미지로 컨테이서 재실행

```yaml
name: Checkit Backend CD # workflow 이름
on: # Event
  pull_request:
    types:
      - closed
    branches: [main, develop]
  workflow_dispatch:

jobs: # Job
  cd: # Job 이름
    if: github.event.pull_request.merged == true # Job 실행 조건

    runs-on: ubuntu-latest # Runner

    steps: # Step
      - name: Checkout
        uses: actions/checkout@v2

      - name: JDK 11을 설치
        uses: actions/setup-java@v3
        with:
          java-version: "11"
          distribution: "temurin"

      # 이미지 빌드를 위해 gradle.properties 생성
      - name: Setting gradle.properties
        run: |
          echo "${{ secrets.GRADLE_PROPERTIES }}" >> gradle.properties
          cat gradle.properties

      # secrets에 base64로 인코딩 되어 있는 application을 디코딩하여 yml파일 생성
      - name: Setting application.yml
        run: |
          echo "${{ secrets.APPLICATION_YML }}" | base64 --decode >> ./src/main/resources/security/application.yml

      # 도커 허브 사용을 위해 도커에 로그인
      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_ID }}
          password: ${{ secrets.DOCKERHUB_TOKEN }} # 도커 홈페이지에서 발급받은 토큰 사용

      # jib로 이미지 빌드 및 푸시
      - name: Build and Push backend docker image using jib
        run: |
          ./gradlew jib

      # ssh를 통해 EC2에 접속, 접속한 후에 실행할 스크립트 작성
      - name: Connect to EC2 using SSH
        uses: appleboy/ssh-action@master
        with:
          host: ${{secrets.AWS_HOST}}
          username: ${{secrets.AWS_USERNAME}}
          key: ${{ secrets.AWS_KEY_PEM }}
          envs: GITHUB_SHA
          script: |
            cd Checkit
            git pull
            sudo docker-compose -f docker-compose.yml down
            sudo docker-compose -f docker-compose.yml pull
            sudo docker-compose -f docker-compose.yml up -d
```

# 결과

### CI/CD를 적용하고 난 후 느낀 장점

1. 배포 자동화
   - 당연한 말이지만 개발 후 이미지를 갱신해주는 것부터 서버에서 컨테이너를 실행 시키는 귀찮은 과정이 없어져서 너무 편하다.
   - 또한 CD 시 이미지를 자동으로 갱신시켜주기 때문에 이미지를 항상 최신으로 유지할 수 있다.
2. 자신감 있는 배포
   - CI를 적용하기 전에는 배포하고 나서 문제를 확인하는 경우가 있어 항상 불안했다.
   - CI 적용 후에는 branch에 merge 됐다는 자체가 코드에 문제가 없다는 것이기 때문에 자신감 있게 배포할 수 있게 되었다.
3. 오류 탐색 시간 단축
   - 여러 사람이 기능 개발을 끝내고 나중에 코드를 확인해보면 갑자기 실행이 안될 때가 있다.
   - CI를 적용하니 어떤 branch에서 무슨 오류가 발생했는지 바로 확인이 가능해 나중에 시간을 들여 오류를 찾는 일이 없어졌다.

CI/CD를 적용하는데 많은 시간이 들었고 어려움이 있었다. 이렇게까지 해서 적용할 필요가 있을까 싶었지만 막상 적용해보니 굉장히 편했다. 더 이상 배포와 오류 탐색에 시간을 들이지 않아도 되니 개발에만 집중할 수 있게 되었다.

# 참고
