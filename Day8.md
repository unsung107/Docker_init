# Day 8



### Docker-compose

도커파일은 하나의 서비스에 대한 것이라고보면됨

공식사이트 : https://docs.docker.com/compose/

```bash
master@ubuntu:~$ docker-compose

Command 'docker-compose' not found, but can be installed with:

sudo snap install docker          # version 19.03.11, or
sudo apt  install docker-compose  # version 1.25.0-1

See 'snap info docker' for additional versions.

master@ubuntu:~$ sudo apt-get -y install docker-compose

```

YAML(야물)

	- 확장자 yml
	- 확장자 yaml

https://docs.docker.com/compose/compose-file/compose-versioning/#version-2

https://docs.docker.com/compose/compose-file/compose-versioning/#version-3

버젼 2와 3가 있다.



```bash
master@ubuntu:~$ mkdir docker-compose
master@ubuntu:~$ cd docker-compose/
master@ubuntu:~/docker-compose$ ls
master@ubuntu:~/docker-compose$ mkdir nginx
master@ubuntu:~/docker-compose$ cd nginx/
master@ubuntu:~/docker-compose/nginx$ vim docker-compose.xml

```

```yaml
# docker-compose.yml
version: '3.1'

services:
  web:
    container_name: mynginx
    image: nginx:latest
    ports:
      - "80:80"
    command: [ 'nginx', '-g', 'daemon off;']
```

```bash
master@ubuntu:~/docker-compose/nginx$ docker-compose up

master@ubuntu:~/docker-compose/nginx$ docker-compose images
master@ubuntu:~/docker-compose/nginx$ docker-compose ps
master@ubuntu:~/docker-compose/nginx$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
c41dd48adf24        bridge              bridge              local
cdf6f4797a5e        host                host                local
f5d0da5036ed        mynetwork           bridge              local
5ff8fb79f574        nginx_default       bridge              local
f515a2d423a5        none                null                local


master@ubuntu:~/docker-compose/nginx$ docker-compose down
Stopping mynginx ... done
Removing mynginx ... done
Removing network nginx_default
# 만들었던 사설 네트워크까지 내려감
```



백그라운드에서 만들자

```bash
master@ubuntu:~/docker-compose/nginx$ docker-compose up -d

master@ubuntu:~/docker-compose/nginx$ docker-compose images
Container   Repository    Tag       Image Id       Size  
---------------------------------------------------------
mynginx     nginx        latest   992e3b7be046   132.9 MB
master@ubuntu:~/docker-compose/nginx$ docker-compose ps
 Name                Command               State         Ports       
---------------------------------------------------------------------
mynginx   /docker-entrypoint.sh ngin ...   Up      0.0.0.0:80->80/tcp
master@ubuntu:~/docker-compose/nginx$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
c41dd48adf24        bridge              bridge              local
cdf6f4797a5e        host                host                local
f5d0da5036ed        mynetwork           bridge              local
ae0ce074bb5e        nginx_default       bridge              local
f515a2d423a5        none                null                local


master@ubuntu:~/docker-compose/nginx$ docker-compose logs
# 로 실행로그확인가ㅡㅇ
master@ubuntu:~/docker-compose/nginx$ docker-compose stop, start #로 중지및 재가동가능


master@ubuntu:~/docker-compose/nginx$ docker-compose exec web /bin/bash # 들어가서관리
```

```bash
mkdir ../mariadb
cd ../mariadb
pwd
: /home/master/docker-compose/mariadb
vim docker-compose.yml
```

```yaml
version: '3.1'

services:
  db:   
    container_name: mymariadb
    image: mariadb:latest
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: example_db
      MYSQL_USER: example_db_user
      MYSQL_PASSWORD: example_db_pass

```

```bash
master@ubuntu:~/docker-compose/mariadb$ docker-compose up -d
master@ubuntu:~/docker-compose/mariadb$ docker-compose ps # 위치조심
  Name                Command             State           Ports         
------------------------------------------------------------------------
mymariadb   docker-entrypoint.sh mysqld   Up      0.0.0.0:3306->3306/tcp

master@ubuntu:~/docker-compose/mariadb$ sudo apt-get -y install mariadb-client
master@ubuntu:~/docker-compose/mariadb$ mysql -h 127.0.0.1 -u root -p # 킁
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4

```

```mysql
MariaDB [(none)]> show databases;
MariaDB [(none)]> select user from mysql.user;
+-----------------+
| User            |
+-----------------+
| example_db_user |
| root            |
| mariadb.sys     |
| root            |
+-----------------+

```



```bash
master@ubuntu:~/docker-compose/mariadb$ docker-compose down

master@ubuntu:~/docker-compose/mariadb$ mkdir ../mariadb-tomcat
master@ubuntu:~/docker-compose/mariadb$ cd ../mariadb-tomcat/

```

```yaml
version: '3.1'

services:
  db:
    container_name: mymariadb
    image: mariadb:latest # ports 가 사라짐 : tomcat 에서 links 를 통해 연결
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: example_db
      MYSQL_USER: example_db_user
      MYSQL_PASSWORD: example_db_pass
  tomcat:
    container_name: mytomcat
    image: tomcat:latest
    ports:
      - "8080:8080"
    links:
      - db 
```

```bash
master@ubuntu:~/docker-compose/mariadb-tomcat$ docker-compose up -d
master@ubuntu:~/docker-compose/mariadb-tomcat$ docker-compose ps
  Name                Command             State           Ports         
------------------------------------------------------------------------
mymariadb   docker-entrypoint.sh mysqld   Up      3306/tcp              
mytomcat    catalina.sh run               Up      0.0.0.0:8080->8080/tcp
master@ubuntu:~/docker-compose/mariadb-tomcat$ mysql -h 127.0.0.1 -u root -o
ERROR 2002 (HY000): Can't connect to MySQL server on '127.0.0.1' (115)
# 내부 연결이라 안됨
master@ubuntu:~/docker-compose/mariadb-tomcat$ curl 127.0.0.1:8080
<!doctype html><html lang="en"><head><title>HTTP Status 404 – Not Found</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1><hr class="line" /><p><b>Type</b> Status Report</p><p><b>Description</b> The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.</p><hr class="line" /><h3>Apache Tomcat/9.0.38</h3></body></html
```

```yaml
  tomcat:
    container_name: mytomcat
    image: tomcat:latest
    ports:
      - "8080:8080"
    volumes:
      - ./webapps:/usr/local/tomcat/webapps
    links:
      - db
# 처럼 볼륨을 통해 외부 소스 사용가능
```



### Docker - 구글클라우드(GCP)로

이제 구글 클라우드에 레지스트리를 만들어봅시다



1. 구글 아이디가 있어야함

2. 결재 등록이 되어있어야함

3. 프로젝트 등록

   https://console.cloud.google.com/apis/dashboard?hl=ko&project=crested-talon-291911&organizationId=01.

   1. api 서비스 - 라이브러리
   2. 클라우드 검색 Cloud Build API 사용

4. cloud build api

   사용자 인증키 생성(json 구조)

   햄버거 - IAM 및 관리자 - 서비스 계정 - 계정만들기

   이름 정하고 - 만들기 -역할:프로젝트소유자

   클릭 - 키추가 json 으로만들면 다운로드됨 - vm웨어로 이동(복붙됨)

1. google cloud sdk 

   https://cloud.google.com/sdk/docs/install?hl=ko

```bash
master@ubuntu:~/docker-compose/mariadb-tomcat$ echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list

master@ubuntu:~/docker-compose/mariadb-tomcat$ sudo apt-get install apt-transport-https ca-certificates gnupg

master@ubuntu:~/docker-compose/mariadb-tomcat$ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -

master@ubuntu:~/docker-compose/mariadb-tomcat$ sudo apt-get update && sudo apt-get install google-cloud-sdk

master@ubuntu:~/docker-compose/mariadb-tomcat$ gcloud version
Google Cloud SDK 313.0.0
alpha 2020.10.02
beta 2020.10.02
bq 2.0.61
core 2020.10.02
gsutil 4.53
kubectl 1.15.11

master@ubuntu:~$ gcloud init
# 로그인

master@ubuntu:~$ sudo cp crested-talon-291911-de54a9491a58.json /usr/lib/google-cloud-sdk/bin/


sudo gcloud auth activate-service-account --key-file=/usr/lib/google-cloud-sdk/bin/crested-talon-291911-de54a9491a58.json

master@ubuntu:~$ gcloud auth configure-docker


```

```bash
vi Dockerfile
[Dockerfile 편집]
FROM ubuntu:latest
MAINTAINER Foo Bar <foo@bar.com>

RUN apt-get update
RUN apt-get -y install vim
[  :wq  ]


master@ubuntu:~/ubuntu$ docker build -t myubuntu .
master@ubuntu:~/ubuntu$ docker tag myubuntu asia.gcr.io/halogen-trilogy-291911/myubuntu
master@ubuntu:~/ubuntu$ docker images
master@ubuntu:~/ubuntu$ docker push asia.gcr.io/halogen-trilogy-291911/myubuntu
```



### Docker Swarm

서버 오케스트레이션

경량형 : docker swarm

중대형 : 쿠버네티스

단순 : ECS



* 중앙집중적ㅇ로 관리
* 스케쥴링, 클러스트링, 서비스 디스커버리, 로깅모니터링
* https://docs.docker.com/engine/swarm



매니저노드 

- 워커노드 (매니저 노드에서 워커노드를 중앙집중적으로 관리한다)
- 워커노드
- 워커노드...

배포단위 : 서비스

## 매니저쪽에서 해야할일

```bash
1. docker swarm init --advertise-addr [ip주소]
#나오는 명령어 복사
2. docker node ls
# 나밖에안나옴
4. docker node ls
# 새로운 워커가 등록됨
5. sudo ls /var/lib/docker/swarm
6. docker service ls
# 등록된 서비스가 없음
# 서비스를 등록시키자 (여러 서비스를 노드 밸런싱해서 돌리겠다)
7. docker service create --name whoami -p 4567:4567 subicura/whoami:1
# 다운받음
8. docker service ps whoami
# 생김
9. curl [node ip]:4567
# 응답함
10. docker service scale whoami=5
# 노드들의 컨테이너 수를 조정하여 서비스 수를 증가시킴
```

## 노드쪽에서

```bash
3. 1.의 명령어 복붙	
```





