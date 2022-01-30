# Dockerize Project

도커(Docker)를 사용해서 실제 프로젝트를 도커화(Dockerize)한다.

## 목차

1. [사전 환경 준비](#1.-사전-환경-준비)
2. [Frontend 도커 이미지 저장](2.-FrondEnd-도커-이미지-저장)
3. [BackEnd 도커 이미지 저장](3.-BackEnd-도커-이미지-저장)
4. [개발용 DB (MySQL) 설치 및 접속](4.-개발용-DB-(MySQL)-설치 및-접속)
5. [번외. docker-compose로 실행](#번외.-docker-compose로-실행)

[참고](#참고)

------

## 1. 사전 환경 준비

- [Docker Install](Docker Install.md) 를 보고 도커 설치한다.

- 프로젝트를 로컬에 저장한다.

  ```sh
  # happyhouse를 예시로 사용
  git clone https://github.com/KJY97/happyhouse.git
  cd happyhouse
  ```

- 도커 명령어는 [이 문서](도커 명령어.md)를 참고한다.

## 2. FrontEnd 도커 이미지 저장

- 로컬에서 FrontEnd 실행 및 웹 브라우저로 접속

  ```sh
  cd fianl_happyhouse_frontend # cd 프로젝트_이름
  npm install
  npm run serve
  ```

- FrontEnd 프로젝트 root 밑에 Dockerfile 생성 **(확장자 없는 Dockerfile)**

  - 해당 프로젝트는 **Vue**를 이용하여 개발했다.

  - Dockerfile 작성 참고 : [Dockerize Vue.js App Docs](https://kr.vuejs.org/v2/cookbook/dockerize-vuejs-app.html)

  - 개발환경이 아닌 운영환경에서 많이 사용하는 **NGINX** 예제 사용 권장

  ```sh
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

- FrontEnd 용 도커 이미지 빌드

  ```sh
  docker build . -t front:0.1
  
  docker images
  ```

  > *front는 image 이름, 0.1은 TAG를 의미한다.*<br/>*`-t` : 도커 이미지 TAG명. 보통 버전 관리 용도로 사용*
  >
  > *도커 이미지 빌드시 `sh: vue-cli-service: not found`가 발생하면 아래와 같은 내용 추가<br/>*
  >
  > ```
  > RUN npm install --production
  > RUN npm install @vue/cli-service
  > ```


- 도커로 FrontEnd 실행 및 웹 브라우저 접속

  ```sh
  docker run -it -p 8080:80 --rm front:0.1
  ```

  > *`--rm` : 컨테이너가 정지되면 자동으로 삭제되는 옵션. 임시테스트용 컨테이너를 실행시켜 보는 용도로 유용함*<br/>*키  `Ctrl + C` : 실행 종료*
  >
  > *주의 : 8080 포트가 사용 중이면 오류 발생함. cmd에서 8080 port 죽이기*

- FrontEnd 확인

  - http://localhost:8080/

## 3. BackEnd 도커 이미지 저장

- 로컬에서 BackEnd 실행 및 확인

  - http://localhost:9999/happyhouse/swagger-ui.html

  ```shell
  cd final_happyhouse
  ./mvnw package # maven Warpper 실행
  java -jar target\final_happyhouse-0.0.1-SNAPSHOT.war
  ```

- BackEnd 프로젝트 root 밑에 Dockerfile 생성 **(확장자 없는 Dockerfile)**

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

- BackEnd 용 도커 이미지 빌드

  ```shell
  docker build . -t back:0.1
  
  docker images
  ```

- 도커로 백엔드 실행 및 웹 브라우저 접속

  - http://localhost:8080/happyhouse/swagger-ui.html

  ```shell
  docker run -it -p 8080:9999 --rm back:0.1
  ```

## 4. 개발용 DB (MySQL) 설치 및 접속

- MySQL 컨테이너 실행 및 실행 여부 조회

  ```shell
  docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD="password" -e MYSQL_DATABASE="dbname" -d mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
  #password: 2216, dbname: ssafy - happyhouse 기준
  
  docker ps
  ```

  > *`-p`: 포트 포워딩 호스트:컨테이너*<br/>
  > *`-e (-env)`: 컨테이너 내에서 사용할 환경변수 설정*<br/>*`-d`: 백그라운드 모드*<br/>*`–name`: 컨테이너 이름*<br/>*`--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci`: 한글이 깨지지 않도록 UTF8 설정*

- MySQL 서버 컨테이너 안에 포함되어 있는 mysql 클라이언트 명령어 실행 및 DB 접속 테스트

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

## 번외. docker-compose로 실행

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

   

## 참고

- Vue Dockerfile : https://kr.vuejs.org/v2/cookbook/dockerize-vuejs-app.html

- Spring Dockerfile : https://spring.io/guides/gs/spring-boot-docker/
- DB : https://poiemaweb.com/docker-mysql