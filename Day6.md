# Day 6

1. IDC 구조
   - 가상화
   - Docker
     - 쿠버네티스
     - System
       - 시스템 구성도
         - 클러스터링 / 분산 / 리플리케이션 ...
         - 마이크로 서비스
     - Network
       - 분할
       - 보안
         - 

2. 네트워킹

   - 두 대 이상의 컨테이너가 통신

     - ex) web server - DB server

     - 통신 방법 1. Link - 구형
     - 통신 방법 2. Bridge network - 신형



### 일단 초기화

```bash
$ docker ps
$ docker ps -a
$ docker rm '모든 컨테이너들'
$ docker images
$ docker rmi '모든 이미지들'
# 초기화 작업 진행
$ docekr volume ls
$ docker volume rm "volume name 전체"
$ docker volume rm $(docker volume ls -qf dangling=true)
: 현재 돌아가고 있는 volume 삭제
$ docker run --help
: link 명령어 참고

```





```bash
$ docker run -it --name ubuntu1 ubuntu
# : 컨테이너 생성
#다른 터미널 생성
$ docker run -it --link ubuntu1:lubuntul --name ubuntu2 ubuntu
# 새로운 터미널 띄우고
$ docker ps
# : 컨테이너 확인
$ docker inspect -f "{{ .HostsPath }}" ubuntu1
#: 호스트파일 경로 출력
master@ubuntu:~$ docker inspect -f "{{.HostsPath }}" ubuntu1
# /home/master/data/docker/containers/af6c85afdf29eed9044891e13e5fa3f00aa6dcf4168bcfa1596a934a6b96e38f/hosts
master@ubuntu:~$ docker inspect -f "{{.HostsPath }}" ubuntu2
# /home/master/data/docker/containers/ce43ed00f00d8cee84dbcaab08fddf777e810fa4ea7cb3b3ef69adae94624ecc/host

sudo cat /home/master/data/docker/containers/af6c85afdf29eed9044891e13e5fa3f00aa6dcf4168bcfa1596a934a6b96e38f/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	af6c85afdf29

sudo cat /home/master/data/docker/containers/ce43ed00f00d8cee84dbcaab08fddf777e810fa4ea7cb3b3ef69adae94624ecc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	lubuntu1 af6c85afdf29 ubuntu1
172.17.0.3	ce43ed00f00d

# in ubuntu1
root@af6c85afdf29:/# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	af6c85afdf29

# in ubuntu2
root@ce43ed00f00d:/# cat etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	lubuntu1 af6c85afdf29 ubuntu1
172.17.0.3	ce43ed00f00d

# in ubuntu2 
root@ce43ed00f00d:/# apt-get update
root@ce43ed00f00d:/# apt-get -y install iputils-ping
root@ce43ed00f00d:/# ping -c 3 lubuntu1

```



이제 마리아 디비를 넣고 포트를 안넣을거고 우분투와 링크로 연결시켜서 접근할거임

```bash
master@ubuntu:~$ docker run -it -d -e MYSQL_ROOT_PASSWORD=123456 --name mariadb1 mariadb

master@ubuntu:~$ docker run -it -d --link mariadb1:lmariadb1 --name ubuntu1 ubuntu

# 호스트 파일 확인
master@ubuntu:~$ docker inspect -f "{{.HostsPath}}" ubuntu1
/home/master/data/docker/containers/3bd95137dccb9e5293bf241501a803db5209de17d22bee8e47f77e14eae17ce4/hosts
master@ubuntu:~$ docker inspect -f "{{.HostsPath}}" mariadb1
/home/master/data/docker/containers/43a2fc32ee6ddcf219952686b36e892fb0b8ae1d4de3a14dc2ccdccbdbeff5ed/hosts

master@ubuntu:~$ sudo cat /home/master/data/docker/containers/3bd95137dccb9e5293bf241501a803db5209de17d22bee8e47f77e14eae17ce4/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	lmariadb1 43a2fc32ee6d mariadb1
172.17.0.3	3bd95137dccb

master@ubuntu:~$ sudo cat /home/master/data/docker/containers/43a2fc32ee6ddcf219952686b36e892fb0b8ae1d4de3a14dc2ccdccbdbeff5ed/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	43a2fc32ee6d

# in ubuntu1
root@ce43ed00f00d:/# apt-get update
root@ce43ed00f00d:/# apt-get -y install mariadb-client # 마리아디비 클라이언트 접속 도구 설치

root@3bd95137dccb:/# mysql -h lmariadb1 -u root -p
Enter password: 

```



두 대 이상의 컨테이너 통신

1. 1

   - web server
   - db server

2. 2

   - db server(replication)

     https://mariadb.com/resources/blog/database-master-slave-replication-in-the-cloud



```bash
$ docker ps
$ docker stop "ubuntu id2자리"
$ docker rm "ubuntu id"
$ docker run -it -d -p 8080:8080 --link mariadb1:lmariadb1 --name tomcat1 tomcat

master@ubuntu:~$ docker run -it -d -p 8080:8080 --link mariadb1:lmariadb1 --name tomcat1 tomcat
812cf2fb21d785152170cdfa2cf5beb2ee32b0a3f58d3eda6592597159e4b173
master@ubuntu:~$ docker exec -it tomcat1 pwd
/usr/local/tomcat
master@ubuntu:~$ docker exec -it tomcat1 mkdir -p ./webapps/ROOT/WEB-INF vi index.jsp
master@ubuntu:~$ vi index.jsp
# jsp 파일 작성후 저장
master@ubuntu:~$ docker cp ./index.jsp tomcat1:/usr/local/tomcat/webapps/ROOT

```

현재 윈도우에 vm 웨어 내에 ubuntu 가 있고 그 안에 톰캣이 있는 상황.

윈도우에서 이 톰캣으로 접근 가능하다.

```bash
master@ubuntu:~$ wget https://downloads.mariadb.com/Connectors/java/connector-java-2.6.2/mariadb-java-client-2.6.2.jar

# maria db클라이언틀 다운로드
master@ubuntu:~$ ls
data     docker     Downloads  mariadb-java-client-2.6.2.jar  Pictures  Templates  website
Desktop  Documents  index.jsp  Music                          Public    Videos
master@ubuntu:~$ docker exec -it tomcat1 mkdir -p ./webapps/ROOT/WEB-INF/lib
master@ubuntu:~$ docker cp ./mariadb-java-client-2.6.2.jar tomcat1:/usr/local/tomcat/webapps/ROOT/WEB-INF/lib
master@ubuntu:~$ vi jdbc.jsp
# 교재에 있는 코드 복붙
master@ubuntu:~$ docker cp ./jdbc.jsp tomcat1:/usr/local/tomcat/webapps/ROOT/

# 이렇게되면 현재 톰캣 컨테이너에는
webapps - ROOT - WEB-INF - lib - jar 파일
			  - index.jsp
			  - jdbc.jsp
# 구조이다

# 후에
master@ubuntu:~$ docker stop tomcat1
master@ubuntu:~$ docker start tomcat1
# 후 http://localhost:8080/jdbc.jsp
현재시각 : 2020-09-24 11:38:45.0 
```

```bash
master@ubuntu:~$ docker network create mynetwork
f5d0da5036ed22b3362770f3ed7098337d7d09e1cff3568ac41674eada7095ca
master@ubuntu:~$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
1603a72a8ec2        bridge              bridge              local
cdf6f4797a5e        host                host                local
f5d0da5036ed        mynetwork           bridge              local
f515a2d423a5        none                null                local

master@ubuntu:~$ docker run -it -d -e MYSQL_ROOT_PASSWORD=123456 --network mynetwork --name mariadb2 mariadb #여기 소속된 네트워크로 연결을 시킬래
master@ubuntu:~$ docker run -it -d -p 8081:8080 --network mynetwork --name tomcat2 tomcat

master@ubuntu:~$ docker exec -it tomcat2 cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.18.0.3	51eb23fc23bc ## maria db와 대역이 똑같다. 끝만다름


master@ubuntu:~$ docker exec -it mariadb2 cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.18.0.2	de6d2ef5b8bd

master@ubuntu:~$ brctl show ## 5일차 확인하기

```



### Dockerfile

이미지 생성용 스크립트

https://docs.docker.com/engine/reference/builder/

```bash
master@ubuntu:~$ docker build --help
```

디렉토리를 만들고 Dockerfile 을만든다(디렉토리당 도커파일 1개)

디렉토리 - Dockerfile

​				- 첨부파일

​				- 서브디렉토리 (X)

```bash
# 컨테이너 모두 지우기

master@ubuntu:~$ docker volume rm $(docker volume ls -qf dangling=true)
# 볼륨 모두 지우기

master@ubuntu:~$ cd docker/
master@ubuntu:~/docker$ ls
master@ubuntu:~/docker$ mkdir ubuntu01
master@ubuntu:~/docker$ cd ubuntu01/
master@ubuntu:~/docker/ubuntu01$ vim Dockerfile
---
FROM ubuntu:latest
MAINTAINER Foo Bar <foo@bar.com>

RUN apt-get update
RUN apt-get -y install vim
---
master@ubuntu:~/docker/ubuntu01$ docker build -t myubuntu1:v0.1 .#(. 필수 > 디렉토리)
master@ubuntu:~/docker/ubuntu01$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
myubuntu1           v0.1                641e1197a643        39 seconds ago      164MB

master@ubuntu:~/docker/ubuntu01$ docker run -it --name myubuntu1 myubuntu1:v0.1
root@c79063e17e12:/# vim
# 설치되어이따
master@ubuntu:~/docker$ cd ubuntu02
master@ubuntu:~/docker/ubuntu02$ ls
master@ubuntu:~/docker/ubuntu02$ vim Dockerfile
---
FROM ubuntu:latest
MAINTAINER Foo Bar <foo@bar.com>

RUN mkdir /data

COPY index.jsp /data
---
master@ubuntu:~/docker/ubuntu02$ docker build -t myubuntu2:v0.1 .
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
<none>              <none>              91ff80348253        About a minute ago   72.9MB

# 도중 실패치 이상한 images가 생김 조심해야함

master@ubuntu:~/docker/ubuntu02$ touch index.jsp
master@ubuntu:~/docker/ubuntu02$ docker build -t myubuntu2:v0.1 .



master@ubuntu:~/docker$ cd ubuntu03/
master@ubuntu:~/docker/ubuntu03$ vim Dockerfile

---
FROM ubuntu:latest
MAINTAINER Foo Bar <foo@bar.com>

VOLUME [ "/data" ]
---
master@ubuntu:~/docker/ubuntu03$ docker build -t myubuntu3:v0.1 .

master@ubuntu:~/docker/ubuntu03$ docker volume ls
DRIVER              VOLUME NAME
# 아무것도 없다

master@ubuntu:~/docker/ubuntu03$ sudo ls /var/lib/docker/volumes
metadata.db

## 컨테이너 생성시 volume 생성
master@ubuntu:~/docker/ubuntu03$ docker run -it --name myubuntu3 myubuntu3:v0.1
master@ubuntu:~$ docker volume ls
DRIVER              VOLUME NAME
local               3850be3be5e3945b8cf6499c8606b1d5c9e640540a0e9fe747a8f04baaf2b506
master@ubuntu:~/docker/ubuntu03$ sudo ls /var/lib/docker/volumes
metadata.db # ...?


# sudo rm -rf /var/lib/docker/volumes 내에 metadata.db 제외하고 모두 삭제하자! 클리어! 컨테이너를 삭제해도 볼륨들은 남아있음 조심1

master@ubuntu:~$ docker run -it --name myubuntu3 -v /home/master/datas:/data myubuntu3:v0.1
root@8e18cf480626:/# cd data/
root@8e18cf480626:/data# ls
root@8e18cf480626:/data# touch test.test
root@8e18cf480626:/data# exit


master@ubuntu:~$ cd datas/
master@ubuntu:~/datas$ ls
test.test

master@ubuntu:~/datas$ docker volume ls
DRIVER              VOLUME NAME
# 아무것도 없다. 직접 연결되어있다.
```

```bash
master@ubuntu:~/docker$ mkdir tomcat01
master@ubuntu:~/docker$ cd tomcat01/
master@ubuntu:~/docker/tomcat01$ ls
master@ubuntu:~/docker/tomcat01$ vim Dockerfile
###################################################
FROM openjdk:11-jdk-slim
MAINTAINER Foo Bar <foo@bar.com>

RUN apt-get update
RUN mkdir -p /usr/share/man/man1
RUN apt-get -y install tomcat9

RUN mkdir -p /usr/share/tomcat9/conf
RUN mkdir -p /usr/share/tomcat9/common/classes
RUN mkdir -p /usr/share/tomcat9/server/classes
RUN mkdir -p /usr/share/tomcat9/shared/classes
RUN mkdir -p /usr/share/tomcat9/temp

RUN cp /etc/tomcat9/server.xml /usr/share/tomcat9/conf/
RUN cp /etc/tomcat9/web.xml /usr/share/tomcat9/conf/
RUN cp /etc/tomcat9/tomcat-users.xml /usr/share/tomcat9/conf/

EXPOSE 8080

CMD /usr/share/tomcat9/bin/catalina.sh run
#######################################################

master@ubuntu:~/docker/tomcat01$ docker build -t mytomcat1:v0.1 .

docker run -it -d -p 8080(호스트):8080(컨테이너) --name mytomcat1 mytomcat1:v0.1

```

