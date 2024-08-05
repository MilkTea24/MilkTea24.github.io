---
title: Docker Compose로 프로젝트 구성하기
author: milktea
date: 2024-08-05 10:00:00 +0800
categories: [Tripko]
tags: [Docker]
pin: true
math: true
mermaid: true
---

# 서론
기존의 프로젝트를 빌드업하기 위해 여러 수정 사항을 도입하기로 하였다.
하지만 기존의 프로젝트는 k8s로 구성되어 있었고 개발 환경에서 설정이 복잡하여 k8s를 Docker Compose로 먼저 마이그레이션 후 개발하기로 결정하였다.
그 과정에서 Docker Compose를 어떻게 작성했는지에 관해 정리한 내용이다.


# Docker Compose?
Docker Compose는 여러 개의 컨테이너로 구동되는 어플리케이션을 정의하고 실행하는데 사용하는 도구이다.
k8s에 비해 간단한 설정으로 여러 개의 컨테이너 관리를 자동화할 수 있다는 장점이 있다.

## Docker Compose의 등장 배경
### 컨테이너 가상화
Docker Compose가 등장한 배경은 무엇일까?
현대의 어플리케이션에서는 컨테이너 가상화 기술이 많이 활용된다.
컨테이너 가상화 기술은 호스트 가상화와 달리 게스트 OS로 인한 오버헤드가 적기 때문에 확장 시 비용이 적다.
따라서 급속히 변화하는 사용자의 트래픽에 대응하기 위해 컨테이너 가상화 + 클라우드가 많이 활용된다.

하지만 현대의 어플리케이션의 규모가 갈수록 커짐으로써 어플리케이션에 모듈화를 도입하게 된다.
예를 들면 DB 컨테이너, 백엔드 컨테이너 등으로 각각의 기능마다 이미지를 만들어 컨테이너를 생성한다.
이 경우 백엔드 컨테이너에 많은 부하가 발생할 경우 새로운 백엔드 컨테이너를 생성하여 부하를 분산할 수 있다.

### 컨테이너 명령어로 생성하기
여러 개의 컨테이너로 구성된 어플리케이션을 실행한다고 가정하자.
이 때 각각의 컨테이너를 아래와 같은 Docker 명령어를 아래처럼 손수 입력하여 실행해야 한다.

```shell
docker run -d \
  --name backend \
  --env-file ./envs/backend.env \
  -p 8080:8080 \
  backend
```

## Docker Compose
Docker compose는 위와 같이 컨테이너 생성을 자동화할 수 있는 도구로 docker-compose.yml에 생성 방법을 정의할 수 있다.
이후 docker-compose.yml 파일을 실행만 하면 정의된 내용을 바탕으로 여러 개의 컨테이너들이 자동으로 생성된다.

Docker Compose는 단일 서버에서 여러 개의 컨테이너를 실행한다.
반면 k8s는 분산된 서버 환경에서 실행되고 scale in/out의 자동화를 지원하므로 실제 배포 환경에서는 k8s가 많이 활용된다.
하지만 개발 환경에서는 설정이 쉬운 Docker Compose를 활용하여 간편하게 개발할 수 있다.

# 현재 프로젝트 Docker Compose 구성하기
## 프로젝트 구조
![img.png](/assets/img/posts/tripko/docker-compose/img2.png)
가장 앞단에는 NGINX 서버가 존재하고 프론트/백엔드 어플리케이션이 존재한다.
백엔드 어플리케이션은 MySQL RDBMS에 데이터를 저장하고 인메모리 저장은 redis를 사용한다.
또한 백엔드 어플리케이션은 amazon S3에서 정적 이미지를 가져오므로 이를 위해 amazon S3 인증 정보를 백엔드 어플리케이션에서 가지고 있어야 한다.

## 필요한 이미지
- NGINX
- 프론트 어플리케이션
- 백엔드 어플리케이션
- MySQL
- Redis

# docker-compose.yml 작성
## 프론트엔드 어플리케이션
프론트엔드 어플리케이션은 우리가 직접 만든 어플리케이션이다.
따라서 docker hub에 이미지가 존재하지 않으므로 직접 빌드해서 이미지를 생성해야 한다.
이미지 만들기 위해서는 DockerFile이 프론트엔드 프로젝트에 존재해야 한다.

### DockerFile
```dockerfile
# 첫번째 스테이지
# 기존의 node:16-alpine3.11 이미지를 바탕
# 별칭을 build로 하여 다른 스테이지에서 참조 가능
FROM node:16-alpine3.11 AS build
# 아래 명령들을 수행할 작업 디렉토리를 /usr/src/app으로 변경
WORKDIR /usr/src/app
# package로 시작하는(package.json, package-lock.json) json 파일을 작업 디렉토리로 복사
COPY package*.json ./
# 의존성 설치
RUN npm ci
# 전체 소스코드를 컨테이너 작업 디렉토리로 복사
COPY . .
# 소스 코드 빌드
RUN npm run build
# 두번째 스테이지
# 기존의 node:16-alpine3.11 이미지를 바탕
FROM node:16-alpine3.11
# 작업 디렉토리 설정
WORKDIR /usr/src/app
# build 스테이지의 파일들을 현재 작업 디렉토리의 build 폴더로 복사
COPY --from=build /usr/src/app/build ./build
# serve 패키지 설치
RUN npm install -g serve
# build 디렉토리 내 정적 파일을 3000번 포트에서 제공하고 single-page 옵션 활성화
CMD ["serve", "-s", "build", "-l", "3000"]
```

### docker-compose.yml
```yaml
services:
  frontend:
    # docker-compose.yml이 위치한 경로에서 상대 경로로 '../../Team6_FE'의 Docker 이미지를 빌드한다.
    build: ../../Team6_FE
    # 환경 변수는 envs 폴더의 frontend.env에 정의된 변수를 사용한다.
    env_file:
      - envs/frontend.env
    # 호스트의 3000번 포트와 컨테이너의 3000번 포트를 매핑한다.
    ports:
      - "3000:3000"
```
외부 트래픽은 호스트의 3000번 포트로 요청하면 컨테이너의 3000번 포트로 매핑된다.
컨테이너의 3000번 포트에는 프론트엔드 어플리케이션이 실행 중이므로 프론트엔드 어플리케이션이 응답할 수 있다.


### frontend.env
```
REACT_APP_API_URL=http://localhost:8080/api
```


## 백엔드 어플리케이션
백엔드 어플리케이션 이미지를 생성하기 위해서 마찬가지로 DockerFile이 있어야 한다.

### Dockerfile
```dockerfile
# gradle:7.6.4-jdk17 이미지를 기반으로 함
FROM openjdk:17-jdk-alpine
# 작업 디렉토리 설정
WORKDIR /home/gradle/project
# Spring 소스 코드를 이미지에 복사
COPY . .
# gradlew를 이용한 프로젝트 필드
RUN ./gradlew clean build
# 빌드 결과 jar 파일을 실행
CMD ["java", "-jar", "-Dspring.profiles.active=prod", "/home/gradle/project/build/libs/tripKo-0.0.1-SNAPSHOT.jar"]
```

### docker-compose.yml
```yaml
# services 내부
  backend:
    # 현재 docker-compose.yml은 백엔드 어플리케이션 내부의 deployment 파일에 있으므로 
    # 상위 폴더의 Docker 이미지를 빌드한다.
    build: ..
    # redis, mysql 서비스 실행 이후 backend 서비스를 실행한다.
    depends_on:
      - redis
      - mysql
    # 환경 변수는 envs 폴더의 backend.env에 정의된 변수를 사용한다.
    env_file:
      - envs/backend.env
    # 호스트의 8080번 포트와 컨테이너의 8080번 포트를 매핑한다.
    ports:
      - "8080:8080"
```

### backend.env
AWS 인증에 필요한 비밀키 등을 env 파일에 관리하고 .env 파일은 .gitignore에 등록함으로써 비밀키의 외부 노출을 막을 수 있다.
```
DATABASE_URL=...
ACCESS_KEY=...
SECRET_KEY=...
```

## MySQL, Redis, NGINX
MySQL과 Redis, NGINX는 docker hub의 이미지를 바탕으로 각각 컨테이너를 생성한다.
백엔드 어플리케이션 컨테이너에 통합하는 대신 데이터베이스와 리버스 프록시 컨테이너를 따로 생성한다.
이 경우 여러 서비스가 독립적인 환경인 컨테이너에서 실행되므로 한 서비스의 장애가 다른 서비스에 영향을 미치지 않는다.
그리고 필요에 따라 개별적으로 확장할 수 있다는 장점도 있다.

### docker-compose.yml: MySQL
```yaml
# services 내부
  mysql:
    ## docker hub의 mysql:8.0.2 이미지를 사용
    image: mysql:8.0.2
    # 다른 컨테이너에서 3306 포트로 접속할 수 있도록 노출한다
    expose:
      - "3306"
    # 환경 변수는 envs 폴더의 mysql.env에 정의된 변수를 사용한다.
    env_file:
      - envs/mysql.env
    # volumes
    volumes:
      # 첫번째 볼륨 설정
      - mysql-data:/var/lib/mysql
      # 두번째 볼륨 설정
      - ./mysql/:/docker-entrypoint-initdb.d/

# services 외부
volumes:
  mysql-data:
```

![img.png](/assets/img/posts/tripko/docker-compose/img-4.png)

기본적으로 하나의 이미지는 여러 레이어로 구성되어 있고 이 레이어들은 불변(Read-Only)이다.
그러면 이미지를 바탕으로 만든 컨테이너에서 데이터를 수정할 때는 어떻게 레이어를 수정할 수 있는지 의문이 들 수 있다.
컨테이너를 실행하면 이미지 레이어들 위에 최상위 레이어인 Thin R/W Layer가 생성되고 이는 수정할 수 있는 레이어이다.
따라서 **변경 사항은 모두 Thin R/W Layer에 저장**된다.

문제는 **이 Thin R/W Layer는 컨테이너가 삭제될 때 같이 삭제가 된다**는 점이다.
영속화해야하는 데이터베이스 내 데이터들이 컨테이너가 삭제될 때 같이 삭제가 된다면 어떻게 될까?
이러한 불상사를 방지하기 위해 Docker의 **볼륨**을 활용할 수 있다.

`mysql-data:/var/lib/mysql`은 mysql-data라는 Docker 볼륨을 컨테이너의 /var/lib/mysql와 연결(마운트)한다.
일반적으로 볼륨은 호스트 머신 내에 따로 저장되므로 컨테이너가 종료되더라도 /var/lib/mysql 내의 데이터는 유지된다.

`./mysql/:/docker-entrypoint-initdb.d/`는 호스트의 mysql 디렉토리를 컨테이너의 /docker-entrypoint-initdb.d/ 디렉토리에 연결한다.
이 디렉토리에 init.sql이 있으면 컨테이너가 실행될 때 자동으로 함께 실행되어 테이블을 생성하거나 초기 데이터를 넣는 작업 등을 할 수 있다.


### docker-compose.yml: Redis
```yaml
  redis:
    # docker hub에서 최신 redis 이미지를 가져와 실행
    image: redis:latest
    # 6379 포트를 노출
    expose:
      - "6379"
```
redis는 캐시 기능을 하므로 데이터를 영속적으로 저장할 필요는 아직 없어 볼륨을 마운트하지 않았다.

### docker-compose.yml: NGINX
```yaml
  nginx:
    depends_on:
      - frontend
      - backend
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./proxy/nginx.conf:/etc/nginx/nginx.conf
```

`./proxy/nginx.conf:/etc/nginx/nginx.conf`는 호스트의 ./proxy/nginx.conf를 /etc/nginx/nginx.conf로 마운트한다는 것이다.
따라서 컨테이너는 호스트 내 ./proxy/nginx.conf를 사용함으로써 원하는 nginx 설정을 미리해 줄 수 있다.

# 결론
## 실행 결과
![img.png](/assets/img/posts/tripko/docker-compose/img-3.png)

![img.png](/assets/img/posts/tripko/docker-compose/img-5.png)

NGINX의 리버스 프록시 사전 설정이 입력되어 localhost로 접속하면 메인 페이지로 이동하고 localhost/api로 접속하면 api가 호출됨을 확인할 수 있다.
또한 backend.env 내의 AWS 시크릿키 등의 환경 변수도 정상 동작하여 이미지들이 Amazon S3에서 가져옴을 확인할 수 있다.

## 결론
docker-compose.yml을 사용하면 단일 인스턴스 내에서 기능 모듈화를 편리하게 하여 안정성을 높일 수 있는 장점이 있다.
그리고 컨테이너를 사전 설정을 바탕으로 자동으로 생성해주므로 개발 단계에서 아주 편리하게 테스트 배포를 수행할 수 있다.
하지만 결국 단일 인스턴스 내에서 실행되므로 급격하게 변화하는 트래픽에 대응하여 본격적인 수평 확장을 수행하기에는 여러 제약이 존재한다.
따라서 배포 단계에서는 Docker Compose보다는 k8s나 여러 클라우드 서비스를 많이 활용하게 된다.


# Reference
[https://docs.docker.com/compose/](https://docs.docker.com/compose/)

[https://docs.docker.com/storage/storagedriver/](https://docs.docker.com/storage/storagedriver/)

[https://docs.docker.com/storage/volumes/](https://docs.docker.com/storage/volumes/)
