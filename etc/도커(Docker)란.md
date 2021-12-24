# 도커란(Docker)

## 목차

- [도커(Docker) 정의](#도커(Docker)-정의)
- [왜 사용할까?](#왜-사용할까?)
- [Windows10 환경에서의 도커(Docker) 설치](#Windows10-환경에서의-도커(Docker)-설치)
- [도커(Docker) 기본 명령어](#도커(Docker)-기본-명령어)
- [프로젝트 도커화](#프로젝트-도커화)
  - [프론트엔드 도커 이미지](#프론트엔드-도커-이미지)
  - [백엔드 도커 이미지](#백엔드-도커-이미지)
- [학습을 위한 참고자료](#학습을-위한-참고자료)

------

## 도커(Docker) 정의

> 2013년에 등장한 리눅스 **컨테이너**를 기반으로 하여 특정한 **서비스를 패키징하고 배포하는데 유용한 오픈소스 프로그램**

- 도커에서 의미하는 **컨테이너**란?

  - 택배나 화물선 위에 수출 용품을 싣는 컨테이너를 대신하여 **프로그램(소프트웨어)을 담는 격리된 공간을** 의미
  - 각각의 격리된 여러개의 소프트웨어 컨테이너에는 ubuntu, centos등의 운영체제, java, python, 아파치 웹서버나, mysql 같은 dbms 등의 프로그램들 까지도 담기게 된다
  - 각 컨테이너는 격리된 공간이기 때문에 한 컨테이너가 해킹을 당하던, 한 컨테이너에 문제가 생기더라도 컨테이너간에 영향을 끼치지 않는다.

  

## 왜 사용할까?

로컬에서 개발한 시스템이 있다. 이 시스템을 서버에 올렸는데 코드가 제대로 동작하지 않는다. <br/>왜냐하면 로컬의 환경은 Windows이고 서버는 Linux이기 때문!! <br/>이와 같은 상황을 **Environment Disparity**라고 한다. 그리고 이러한 **개발 환경이 맞지 않는 상태를 도커(Docker)는 해결 할 수 있다**.

또한 매번 새로운 버전 출시로 인한 버전이슈 문제도 도커(Docker)는 효율적으로 개선시켜 준다.

결국 도커(Docker)를 사용하면 개발과정에서 필요한 환경 구성이 편리해지고 여러 가지 서비스를 레고 블럭처럼 쌓아서 매쉬업 하기도 용이하기 때문에 도커(Docker)를 사용하는 것이다.



## Windows10 환경에서의 도커(Docker) 설치

1. 컴퓨터 하드웨어의 BIOS 레벨에서 **가상화(Virtualization)** 지원 및 활성화

   - `작업 관리자 > 성능`에서 **'가상화: 사용'** 으로 되어 있는지 확인한 다음, `제어판 > Window 기능 켜기/끄기`에서 **Hyper-V** 체크 확인 후 **리부팅**
   - **'가상화: 사용 안 함'**으로 되어 있다면, 컴퓨터를 재부팅하고 **BIOS**로 진입하여 해당 옵션을 **Enabled**로 설정한 다음 다시 확인한다.

2. 사이트에서 OS에 맞는 버전 설치

   - https://www.docker.com/products/docker-desktop

3. 도커(Docker)를 사용하기 위해 **회원 가입** 진행

4. **PowerShell**을 통해 도커(Docker) 설치 확인

   - 실행 창 열고 PowerShell 검색

   ```shell
   # 버전 확인
   docker -v
   
   # 테스트용 Hello world 도커 컨테이너 실행
   docker run hello-world
   ```

   > *hello world 도커 이미지가 자동 다운로드되어 실행된 결과로 "Hello from Docker!"가 포함된 메시지가 출력되면 성공!*

- 도커실행 시 **WSL2 관련 오류 메시지** 나온다면 WSL2 설치



## 도커(Docker) 기본 명령어

1. 컨테이너 조회

   - 실행중인 컨테이너 조회

     ```shell
     docker ps
     ```

   - 정지(실행 종료)된 컨테이너까지 조회

     ```shell
     docker ps -a
     ```

2. Hello world 컨테이너 삭제

   ```shell
   docker rm [컨테이너 ID 또는 NAME]
   ```

   > *컨테이너 실행 시 `docker run --name=[컨테이너 이름] ~` 과 같이 --name 옵션을 추가하면 자동으로 만들어진 이름 대신 지정된 이름으로 생성 가능*.
   >
   > *컨테이너 ID는 앞 일부분만 입력 가능*.

3. 도커 이미지 조회

   ```shell
   docker images
   ```

4. Hello world 도커 이미지 삭제

   ```shell
   docker rmi [이미지 ID 또는 이미지명:TAG명]
   ```

   > *TAG 중에 latest 태그명은 생략 가능*.



## 프로젝트 도커화

> 도커(Docker)를 사용해서 실제 프로젝트를 도커화(Dockerize)한다.

**프로젝트 코드 다운로드**

```shell
# happyhouse를 예시로 사용
git clone https://github.com/KJY97/happyhouse.git
cd happyhouse
```

### 프론트엔드 도커 이미지

1. 로컬에서 프론트엔드 실행 및 웹 브라우저로 접속

   ```shell
   cd fianl_happyhouse_frontend # cd 프로젝트_이름
   npm install
   npm run serve
   ```

2. 프론트엔드 프로젝트 root 밑에 Dockerfile 생성 **(확장자 없는 Dockerfile)**

   - 해당 프로젝트는 **Vue**를 이용하여 개발했다.
   - Dockerfile 작성 참고 : [Dockerize Vue.js App Docs](https://kr.vuejs.org/v2/cookbook/dockerize-vuejs-app.html)
   - 개발환경이 아닌 운영환경에서 많이 사용하는 NGINX 예제 사용 권장

```shell
# build stage
FROM node:lts-alpine as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
RUN npm run build

# production stage
FROM nginx:stable-alpine as production-stage
COPY --from=build-stage /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

3. 프론트엔드 용 도커 이미지 빌드

```shell
docker build . -t front:0.1

docker images
```

> *front는 image 이름, 0.1은 TAG를 의미한다.*<br/>*-t : 도커 이미지 TAG명. 보통 버전 관리 용도로 사용*
>
> *도커 이미지 빌드시 `sh: vue-cli-service: not found`가 발생하면 아래와 같은 내용 추가<br/>*
>
> ```
> RUN npm install --production
> RUN npm install @vue/cli-service
> ```

4. 도커로 프론트엔드 실행 및 웹 브라우저 접속

```shell
docker run -it -p 8080:80 --rm front:0.1
```

> *--rm : 컨테이너가 정지되면 자동으로 삭제되는 옵션. 임시테스트용 컨테이너를 실행시켜 보는 용도로 유용함*<br/>*키  `Ctrl + C` : 실행 종료*
>
> *주의 : 8080 포트가 사용 중이면 오류 발생함. cmd에서 8080 port 죽이기*

### 백엔드 도커 이미지

1. 로컬에서 백엔드 실행 및 확인

   - http://localhost:9999/happyhouse/swagger-ui.html

   ```shell
cd final_happyhouse
   ./mvnw package
java -jar target\final_happyhouse-0.0.1-SNAPSHOT.war

2. 백엔드 프로젝트 root 밑에 Dockerfile 생성 **(확장자 없는 Dockerfile)**

   - 해당 프로젝트는 Spring boot를 이용하여 개발했다.
   - Dockerfile 작성 참고 : [Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/)

   ```shell
   # openjdk 8 위에서 실행된다는 뜻
   FROM openjdk:8-jdk-alpine 
   # JAR_FILE을 이용해 어떤 어플리케이션의 실행파일을 연결한다
   ARG JAR_FILE=target/*.war
   # 도커 컨테이너 내부에서는 9999 포트를 가지고 돈다
   EXPOSE 9999
   # JAR_FILE를 이름으로 복사
   COPY ${JAR_FILE} app.war
   # Run the jar file
   ENTRYPOINT ["java","-jar","/app.war"]
   ```

3. 백엔드 용 도커 이미지 빌드

   ```shell
   docker build . -t back:0.1
   
   docker images
   ```

4. 도커로 백엔드 실행 및 웹 브라우저 접속

   - http://localhost:8080/happyhouse/swagger-ui.html

   ```shell
   docker run -it -p 8080:9999 --rm back:0.1
   ```

### MySQL 설치 및 접속

1. 개발용 DB로 사용할 MySQL 컨테이너 실행 및 실행 여부 조회

   ```shell
   docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD="password" -e MYSQL_DATABASE="dbname" -d mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
   #password: 2216, dbname: ssafy - happyhouse 기준
   
   docker ps
   ```

   > *-p: 포트 포워딩 호스트:컨테이너*<br/>
   > *-e (-env): 컨테이너 내에서 사용할 환경변수 설정*<br/>*-d: 백그라운드 모드*<br/>*–name: 컨테이너 이름*<br/>*--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci: 한글이 깨지지 않도록 UTF8 설정*

2. MySQL 서버 컨테이너 안에 포함되어 있는 mysql 클라이언트 명령어 실행 및 DB 접속 테스트

   ```shell
   docker exec -it mysql bash # mysql 실행
   mysql -uroot -p ssafy # DB 접속
   show databases; # 현재 계정이 접근 가능한 DB 목록
   show tables; # 현재 DB의 table 목록
   exit # 접속 종료
   ```

   > *docker exec -it mysql mysql -uroot -p ssafy : 한줄로 실행할 수 있다.*
   >
   > *주의: `Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'`가 발생하면 다음 코드 실행<br/> `service mysql restart`*

### docker-compose 실행

> 프론트엔드, 백엔드, DB까지 포함시켜 한번에 실행해보기

- docker 실행 옵션을 미리 작성하고 편하게 사용하기 위함
- frontend, backend, DB, webserver를 각각 컨테이너로 사용했다

1. `docker-compose.yml` 파일 생성

   ```yml
   version: "3" # 파일 규격 버전
   services: # 이 항목 밑에 실행하려는 컨테이너 들을 정의
     frontend:
       image: front:0.1
       build: 
         context: ./fianl_happyhouse_frontend
         dockerfile: Dockerfile
       ports:
         - "8080:8080"
         
     nginx:
       build:
         dockerfile: Dockerfile
         context: ./nginx
       port:
         - "80:80"
       depends_on:
         - frontend
         
     backend:
       image: back:0.1
       build: 
         context: ./final_happyhouse
         dockerfile: Dockerfile
       ports:
         - "9999:9999"
         
     db: # 서비스 명
       image: mysql # 사용할 이미지
       restart: always
       container_name: mysql # 컨테이너 이름 설정
       ports:
         - "3306:3306" # 접근 포트 설정 (컨테이너 외부:컨테이너 내부)
       environment: # -e 옵션
         MYSQL_ROOT_PASSWORD: 2216  # MYSQL 패스워드 설정 옵션
         MYSQL_DATABASE: ssafy
       command: # 명령어 실행
         - --character-set-server=utf8mb4
         - --collation-server=utf8mb4_unicode_ci
   ```

2. docker-compose 파일 실행

   ```shell
   docker-compose up -d
   ```

   

## 학습을 위한 참고자료

- https://www.youtube.com/playlist?list=PLApuRlvrZKogb78kKq1wRvrjg1VMwYrvi [따배도 도커 시리즈]
- https://www.44bits.io/ko/post/easy-deploy-with-docker [도커 튜토리얼]
- https://hub.docker.com/_/mysql [도커_mysql]
- https://m.blog.naver.com/hj_kim97/222315587407 [도커_mysql 사용]
- https://dongyeopgu.github.io/server/docker-compose-%EC%9D%B4%EC%9A%A9%ED%95%98%EA%B8%B0.html [docker-compose]

