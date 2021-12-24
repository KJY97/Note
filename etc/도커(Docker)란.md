# 도커(Docker)

> 2013년에 등장한 리눅스 컨테이너를 기반으로 하여 특정한 서비스를 패키징하고 배포하는데 유용한 오픈소스 프로그램

## 왜 사용할까?

로컬에서 개발한 시스템이 있다. 이 시스템을 서버에 올렸는데 코드가 제대로 동작하지 않는다. <br/>왜냐하면 로컬의 환경은 Windows이고 서버는 Linux이기 때문!! <br/>이와 같은 상황을 **Environment Disparity**라고 한다. 그리고 이러한 **개발 환경이 맞지 않는 상태를 도커(Docker)는 해결 할 수 있다**.

또한 매번 새로운 버전 출시로 인한 버전이슈 문제도 도커(Docker)는 효율적으로 개선시켜 준다.

결국 도커(Docker)를 사용하면 개발과정에서 필요한 환경 구성이 편리해지고 여러 가지 서비스를 레고 블럭처럼 쌓아서 매쉬업 하기도 용이하기 때문에 도커(Docker)를 사용하는 것이다.



## Windows 환경에서의 도커(Docker) 설치

1. 컴퓨터 하드웨어의 BIOS 레벨에서 **가상화(Virtualization)** 지원 및 활성화

   - 작업 관리자 > 성능에서 **'가상화: 사용'** 으로 되어 있는지 확인
   - 사용 안 함으로 되어 있다면, 컴퓨터를 재부팅하고 BIOS로 진입하여 해당 옵션을 Enabled로 설정한 다음 다시 확인한다.

2. 사이트에서 OS에 맞는 버전 설치

   - https://www.docker.com/products/docker-desktop

3. **PowerShell**을 통해 도커(Docker) 설치 확인

   - 실행 창 열고 PowerShell 검색

   ```shell
   # 버전 확인
   docker -v
   
   # 테스트용 Hello world 도커 컨테이너 실행
   docker run hello-world
   ```

   > hello world 도커 이미지가 자동 다운로드되어 실행된 결과로 "Hello from Docker!"가 포함된 메시지가 출력되면 성공!



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

   > - *컨테이너 실행 시 `docker run --name=[컨테이너 이름] ~` 과 같이 --name 옵션을 추가하면 자동으로 만들어진 이름 대신 지정된 이름으로 생성 가능*
   > - *컨테이너 ID는 앞 일부분만 입력 가능*

3. 도커 이미지 조회

   ```shell
   docker images
   ```

4. Hello world 도커 이미지 삭제

   ```shell
   docker rmi [이미지 ID 또는 이미지명:TAG명]
   ```

   > TAG 중에 latest 태그명은 생략 가능



## 프로젝트 도커화

도커(Docker)를 사용해서 실제 프로젝트를 도커화(Dockerize)한다.

프로젝트 코드 다운로드

```shell
git clone https://lab.ssafy.com/s06-study/s06p10b158.git
cd s06p10b158
```

### 프론트엔드 도커 이미지

1. 로컬에서 프론트엔드 실행 및 웹 브라우저로 접속

   ```shell
   cd frontend
   npm install
   npm run serve
   ```

2. 프론트엔드 프로젝트 root 밑에 Dockerfile 생성

   - 해당 프로젝트는 Vue를 이용하여 개발했다.
   - Dockerfile 작성 참고 : [Dockerize Vue.js App Docs](https://kr.vuejs.org/v2/cookbook/dockerize-vuejs-app.html)

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

> *front는 image 이름, 0.1은 TAG를 의미한다.*

4. 도커로 프론트엔드 실행 및 웹 브라우저 접속

```shell
docker run -it -p 8080:80 --rm front:0.1
```



### 백엔드 도커 이미지

1. 로컬에서 백엔드 실행 및 확인

   - http://localhost:9999/happyhouse/swagger-ui.html

   ```shell
   cd backend\final_happyhouse
   ./mvnw package
   java -jar target\final_happyhouse-0.0.1-SNAPSHOT.war
   ```

2. 백엔드 프로젝트 root 밑에 Dockerfile 생성

   - 해당 프로젝트는 Spring boot를 이용하여 개발했다.
   - Dockerfile 작성 참고 : [Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/)

   ```shell
   FROM openjdk:8-jdk-alpine
   ARG JAR_FILE=target/*.war
   EXPOSE 9999
   COPY ${JAR_FILE} app.war
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

5. 개발용 DB로 사용할 MySQL 컨테이너 실행 및 실행 여부 조회

   ```shell
   docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=ssafyroom -e MYSQL_DATABASE=ssafy -d mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
   
   docker ps
   ```

   > *-p: 포트 포워딩 호스트:컨테이너*<br/>
   > *-e: 컨테이너 내에서 사용할 환경변수 설정*<br/>*-d: 백그라운드 모드*<br/>*–name: 컨테이너 이름*<br/>*--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci: 한글이 깨지지 않도록 UTF8 설정*

   - 위 명령어를 실행하여 컨테이너를 올려도 되지만 아래와 같이 docker-compose.yml 파일로 만들어서 실행할 수 있다.

     5-1. docker-compose.yml 파일 생성

     ```shell
     version: "3" # 파일 규격 버전
     services: # 이 항목 밑에 실행하려는 컨테이너 들을 정의
       db: # 서비스 명
         image: mysql # 사용할 이미지
         container_name: mysql # 컨테이너 이름 설정
         ports:
           - "3306:3306" # 접근 포트 설정 (컨테이너 외부:컨테이너 내부)
         environment: # -e 옵션
           MYSQL_ROOT_PASSWORD: "password"  # MYSQL 패스워드 설정 옵션
         command: # 명령어 실행
           - --character-set-server=utf8mb4
           - --collation-server=utf8mb4_unicode_ci
     ```

     5-2. docker-compose 파일 실행

     ```shell
     docker-compose up -d
     ```

6. MySQL 서버 컨테이너 안에 포함되어 있는 mysql 클라이언트 명령어 실행 및 DB 접속 테스트

   ```shell
   docker exec -it mysql
   mysql -uroot -p ssafy
   ```

   

## 학습을 위한 참고자료

- https://www.youtube.com/playlist?list=PLApuRlvrZKogb78kKq1wRvrjg1VMwYrvi
- https://www.44bits.io/ko/post/easy-deploy-with-docker
- https://hub.docker.com/_/mysql

