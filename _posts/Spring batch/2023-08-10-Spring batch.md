---
title: Spring batch 알아보기
author: jimin
date: 2023-08-10 00:00:00 +0900
categories: [Java, Spring batch]
tags: [Spring, Batch]
pin: false
---

# Batch Application

- Batch는 **일괄처리**를 의미
- 매일 전 날의 데이터를 집계 해야한다고 가정
  1. 전날의 데이터를 읽어온다.
  2. 읽어온 데이터를 가공한다.
  3. 가공한 데이터를 저장한다.
- 만약 앱 어플리케이션에서 데이터를 읽고, 가공하고, 저장한다면 해당 서버는 데이터 집계를 위해 많은 서버 자원을 사용해야 함 → 이는 앱 어플리케이션에서 원래 처리해줘야 하는 request를 처리하지 못하게 됨
- 따라서 이런 대용량의 데이터를 처리해주는 어플리케이션이 **Batch Application**이다**.**

### 배치 애플리케이션은 다음의 조건을 만족해야 함

- **대용량 데이터** : 대량의 데이터를 가져오거나, 전달하거나, 계산하는 등 특정 로직을 처리 할 수 있어야 한다.
- **자동화** : 심각한 문제 해결을 제외하고는 사용자 개입 없이 실행되어야 한다.
- **견고성** : 잘못된 데이터를 충돌/중단 없이 처리할 수 있어야 한다.
- **신뢰성** : 무엇이 잘못되었는지를 추적할 수 있어야 한다. (로깅, 알림)
- **성능** : 지정한 시간 안에 처리를 완료하거나 동시에 실행되는 다른 애플리케이션을 방해하지 않도록 수행되어야 한다.

# Spring Batch란?

- Spring 진영에서 지원하는 Batch Framework
- 대용량 일괄처리의 편의를 위해 설계된 가볍고 포괄적임
- Spring의 특성을 그대로 가져왔기 때문에 DI, AOP, 서비스 추상화 등 Spring 프레임워크의 3대 요소를 모두 사용 가능
- Spring Batch를 사용하는 경우
  - 대용량의 비즈니스 데이터를 복잡한 작업으로 처리해야하는 경우
  - 특정한 시점에 스케쥴러를 통해 자동화된 작업이 필요한 경우 (ex. 푸시알림, 월 별 리포트)
  - 대용량 데이터의 포맷을 변경, 유효성 검사 등의 작업을 트랜잭션 안에서 처리 후 기록해야하는 경우

## 구성요소

- Spring Batch는 Job, Step으로 구성되어 있다.

![구성요소](/assets/img/postpic/Springbatch/batch%20구성요소.png)

### Job

- 하나 이상의 Step으로 구성된 단위적인 배치 작업 단위
- 각 Job은 개별적으로 실행, 일련의 단계를 순서대로 처리
- 배치 작업 중 Step이 실패하거나 중단된 경우에도 재시작 가능

**JobInstance**

- Job의 실행 단위
- 1일과 2일, 총 2번 실행 시 각각의 JobInstance가 생성

**JobParameters**

- JobInstance를 구별할 때 사용
- String, Double, Long, Date 4가지 형식을 지원

**JobExecution**

- JobInstance에 대한 실행 시도에 대한 객체
- 실패하여 재실행 시킨 경우 동일한 JobInstance이나 2번 실행에 대한 JobExecution은 개별로 생성
- JobInstanced 실행에 대한 상태, 시작시간, 종료시간, 생성시간 등에 대한 정보 저장

**JobRepository**

- JobRepository는 위에서 말한 모든 배치 처리 정보를 담고있는 매커니즘
- Job이 실행되게 되면 JobRepository에 JobExecution과 StepExecution을 생성하게 되며 JobRepository에서 Execution 정보들을 저장하고 조회하며 사용

**JobLauncher**

- JobLauncher는 Job과 JobParameters를 사용하여 Job을 실행하는 객체

### Step

- Job 안에 있는 단일 작업 단위
- Job은 일련의 단계, 각 단계가 Step을 의미
- 여러 단계로 구성된 복잡한 배치 작업을 더 작은 단위로 나누어 관리 가능
- 일반적으로 데이터 읽기, 처리, 쓰기와 같은 작업이 하나의 Step
- chunk 지향 Step과 tasklet을 통해 생성 가능, 이 둘은 같이 사용 x

**StepExecution**

- JobExecution과 동일하게 Step 실행 시도에 대한 객체를 의미
- 이전 단계의 Step이 실패하게 되면 실패 이후 StepExecution은 생성 x
- JobExecution과 동일하게 실제 시작이 될 때만 생성
- JobExecution에 저장되는 정보 외에 read 수, write 수, commit 수, skip 수 등의 정보들도 저장

**ExecutionContext**

- Job에서 데이터를 공유 할 수 있는 데이터 저장소
- ExecutionContext를 통해 Step간 Data 공유가 가능
- Job 실패시 ExecutionContext를 통한 마지막 실행 값을 재구성 가능
- 2가지 ExecutionContext이 존재 (JobExecutionContext, StepExecutionContext)
  - JobExecutionContext의 경우 Commit 시점에 저장
  - StepExecutionContext는 실행 사이에 저장

### **Chunk 지향 Step**

- 데이터를 청크(chunk) 단위로 처리하는 방식
- 데이터를 작은 묶음으로 나누어 처리하므로 대량의 데이터를 처리하기에 적합
- 트랜잭션 내에서 처리되기 때문에 실패하더라도 롤백, 안정성을 보장
- 일반적으로 읽기, 처리, 쓰기 단계를 통해 데이터를 처리
- JDBC, Hibernate, JPA를 지원
  **ItemReader(읽기)**
  - 데이터를 읽어오는 인터페이스
  - 파일, 데이터베이스, 메시지 큐 등으로부터 데이터를 읽고 Chunk 단위로 처리할 아이템들을 반환
  - read() 메소드를 정의하고 있으며, 이를 Override 하여 커스텀 가능
    **ItemWriter(쓰기)**
  - 아이템들을 최종적으로 저장하거나 출력하는 역할을 담당하는 인터페이스
  - ItemReader 또는 ItemProcessor를 통해 가공된 아이템들을 사용
  - write() 메서드를 정의하고 있으며, 이를 Override 하여 커스텀 가능
    **ItemProcessor(처리)**
  - 읽어온 아이템을 가공하거나 변환하는 역할을 담당하는 인터페이스
  - ItemReader에서 읽어온 원본 데이터를 가공하여 다른 형태로 변환하거나, 필터링하거나, 유효성 검사 등의 작업을 수행
  - 배치를 처리하는데 필수 요소는 x
  - process() 메서드를 정의하고 있으며, 이를 Override 하여 커스텀 가능

### Tasklet

- 하나의 메서드로 구성 되어있는 간단한 인터페이스, 비교적 쉽기 때문에 간단한 처리를 위해 사용
- 하나의 단위 작업만 처리
- Step의 실행 전후로 특정한 처리를 수행하거나, 특정 조건에 따라 Step의 실행 여부를 결정

# Spring Batch Schema

- 스프링 배치의 실행 및 관리를 위한 목적으로 여러 도메인들(Job, Step, JobParameters 등)에 대한 정보를 저장, 업데이트, 조회 등의 작업을 할 수 있는 스키마
- 과거 및 현재의 **Batch 실행에 대한 정보**(성공, 실패 여부)를 관리함으로서 운영 및 장애와 관련된 대처 가능
- 내장 메모리를 사용하지 않고 DB와 연동할 경우 필수

![스키마](/assets/img/postpic/Springbatch/batch%20구성요소.png)

**BATCH_JOB_INSTANCE**

- Job이 실행될 때 JobInstance 정보가 저장
- 실행되는 Job의 Name, Key에 대한 데이터를 저장

**BATCH_JOB_EXECUTION**

- Job의 실행 정보가 저장되며 Job의 생성, 시작, 종료 시각, 실행상태, 메시지 등을 관리

**BATCH_JOB_EXECUTION_PARAMS**

- Job과 함께 실행되는 JobParameters 정보를 저장

**BATCH_JOB_EXECUTION_CONTEXT**

- Job의 실행동안 여러 상태정보, 공유 데이터를 직렬화(JSON)해서 저장
- Step 간 데이터 공유가 가능

**BATCH_STEP_EXECUTION**

- Step의 실행 정보가 저장되며 Step의 생성, 시작, 종료 시각, 실행상태, 메시지 등을 관리

**BATCH_STEP_EXECUTION_CONTEXT**

- Step의 실행동안 여러 상태정보, 공유 데이터를 직렬화(JSON)해서 저장
- Step별로 저장되며 Step간 데이터를 공유 x

# 참고

- [https://khj93.tistory.com/entry/Spring-Batch란-이해하고-사용하기](https://khj93.tistory.com/entry/Spring-Batch%EB%9E%80-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B3%A0-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
- https://jojoldu.tistory.com/324
- https://azderica.github.io/01-spring-batch/
- https://zzang9ha.tistory.com/426
