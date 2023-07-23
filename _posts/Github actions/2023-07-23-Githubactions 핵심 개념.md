---
title: Github Actions 핵심 개념 & 간단한 CI/CD 적용
author: jimin
date: 2023-07-23 00:00:00 +0900
categories: [Github_Actions]
tags: [Github_Actions, AWS, CI/CD]
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

# 실습

### 이슈 태그 생성

```yaml
name: issue tagging  # Workflow name
on:  # Event
  issues:
    types:
      - reopened
      - opened

jobs:  # Job
  label_issues:  # Job name
    runs-on: ubuntu-latest  # Runner 
    permissions:
      issues: write
    steps:  # Step
      - uses: actions/github-script@v6  # action
        with:
          script: |
            github.rest.issues.addLabels({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              labels: ["good"]
            })
```

### CI

```yaml
name: Django CI # workflow 이름

on: # Event
  push:
    branches:
      - main
#     - develop (다른 브랜치도 추가 가능)
  pull_request:
    branches:
      - main

jobs: # Job
  ci: # Job 이름
    runs-on: ubuntu-latest # Runner

    services: # 컨테이너, docker-compose 설정과 거의 유사
      db:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: ${{ secrets.MYSQL_ROOT_PASSWORD }} # github에 등록한 환경변수
          MYSQL_DATABASE: ${{ secrets.MYSQL_DATABASE }}
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps: # Step
    # 레포지토리의 소스 코드를 체크아웃하여 작업 디렉토리로 가져오는 action
    - name: Checkout
      uses: actions/checkout@v2 

    # mysql 컨테이너 연결 확인
    - name: Verify MySQL connection
      run: |
        mysql --version
        mysql --host 127.0.0.1 --port 3306 -u ${{ secrets.MYSQL_USERNAME }} -p${{ secrets.MYSQL_ROOT_PASSWORD }}

    # 파이썬 3.11.0 버전 설치
    - name: Set up Python 3.11.0 
      uses: actions/setup-python@v2
      with:
        python-version: 3.11.0 

    # .env 생성
    - name: Setting .env
      run: |
          echo "${{ secrets.ENV }}" >> .env
          cat .env

    # 의존성 설치
    - name: Install Dependencies
      run: |
        python -m pip install --upgrade pip
        cd github_actions
        pip install -r requirements.txt

    # mysql 컨테이너에 migration, 테이블 생성
    - name: Run migrations
      run: |
        cd github_actions
        python manage.py makemigrations --settings=config.settings.actions
        python manage.py migrate --settings=config.settings.actions

    # 테스트 진행
    - name: Run Tests
      run: |
        cd github_actions
        mysql --host 127.0.0.1 --port 3306 -u ${{ secrets.MYSQL_USERNAME }} -p${{ secrets.MYSQL_ROOT_PASSWORD }}
        python manage.py test --settings=config.settings.actions
```

### CD

```yaml
name: Django CD  # workflow 이름
on: # Event
  pull_request:
    types:
      - closed
    branches: [ main ]

jobs: # Job
  cd: # Job 이름
    if: github.event.pull_request.merged == true # Job 실행 조건

    runs-on: ubuntu-latest # Runner

    steps: # Step
      - name: Checkout
        uses: actions/checkout@v2

      - name: Set up Python 3.11.0 
        uses: actions/setup-python@v2
        with:
          python-version: 3.11.0 

      - name: Install Dependencies
        run: |
          python -m pip install --upgrade pip
          cd github_actions
          pip install -r requirements.txt

      # 도커 허브 사용을 위해 도커에 로그인
      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }} # 도커 홈페이지에서 발급받은 토큰 사용

      # 이미지 빌드를 위해 .env 생성
      - name: Setting .env
        run: |
          echo "${{ secrets.ENV }}" >> .env
          cat .env

      # docker-compose에서 사용하는 이미지들을 build, 공식 이미지들은 x
      - name: Build docker images
        run: |
          docker-compose -f docker-compose.prod.yml build
          docker images

      # docker hub repository에 이미지들을 push
      - name: Push docker images 
        run: docker-compose -f docker-compose.prod.yml push

      # ssh를 통해 EC2에 접속, 접속한 후에 실행할 스크립트 작성
      - name: Connect to EC2 using SSH
        uses: appleboy/ssh-action@master
        with:
          host: ${{secrets.AWS_HOST}}
          username: ${{secrets.AWS_USERNAME}}
          key: ${{ secrets.AWS_KEY_PEM }}
          envs: GITHUB_SHA
          script: |
              cd github_actions_session
              git pull
              sudo docker-compose -f docker-compose.prod.yml down
              sudo docker-compose -f docker-compose.prod.yml pull
              sudo docker-compose -f docker-compose.prod.yml up -d
```

# Github Repository

- [https://github.com/jiminsong490/github_actions_session.git](https://github.com/jiminsong490/github_actions_session.git)

## 서버 환경

- t2.small
- ubuntu 22.04
- docker 설치, docker-compose 2버전 이상 설치
- git clone 및 .env 파일 생성되어있음
- 이미 docker-compose가 실행되고 있는 상황
- 보안 그룹 인바운드 : 22, 8000번 모든 ip 접속 가능

# 참고

- [https://docs.github.com/ko/actions/learn-github-actions/understanding-github-actions](https://docs.github.com/ko/actions/learn-github-actions/understanding-github-actions)

- [https://www.notion.so/django_docker-f09219e62f164f48947312cc40899342](https://www.notion.so/f09219e62f164f48947312cc40899342?pvs=21)

- [https://seosh817.tistory.com/104](https://seosh817.tistory.com/104)