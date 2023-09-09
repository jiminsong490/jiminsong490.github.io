---
title: Docker hub + Github actions(CI/CD)
author: jimin
date: 2023-08-20 00:00:00 +0900
categories: [Github_Actions]
tags: [Github_Actions, Spring_boot, CI/CD]
pin: false
---

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
