# Day 4

## 시스템구조

이미지 저장 구조

 :  도커의 os 이미지 -> Kernal 이 빠진, 쉘만있음



1. 도커 클라이언트 : CLI

2. 도커 서버

   데몬으로 실행됨

   ```bash
   master@ubuntu:~$ sudo systemctl status docker 로 확인가능 (exit : q)
   
   master@ubuntu:~$ cat /lib/systemd/system/docker.service  #실행 환경에 대한 설정
   
   ```



1. 도커 클라이언트

- 디렉토리 (자료) : /var/lib/docker
  - containers - container
  - image
    	- imagedb
    	- layerdb - overlay2
  - overlay2
     - 이미지 정보
       * 증분 백업 처럼 들어있다.한번에 다들어있지않고 나뉘어서들어가있음(layer)



2. 포트 연결 정보(단독서버)

   jdk -> tomcat

```bash
master@ubuntu:~/docker$ docker run -it --name tomcat1 -p 8080:8080 openjdk:11-jre-slim /bin/bash

# apt-get update
# mkdir -p /usr/shar/man/man1 을 추가하면 day 3에서 톰캣 설치시에 나온 에러 안나옴
# apt-get -y install tomcat9
```



---



### 시스템 관리 명령어

```bash
$ docker system --help

$ docker system df
$ docker system info == docker info
```

### 프로세스 정보

```bash
$ docker ps
$ docker ps -a
$ docker container ls
$ docker container ls -a
```

### 리소스 사용량 명령어

```bash
$ docker stats --help

```



### 실습

```bash
master@ubuntu:~$ docker run -it --name ubuntu1 ubuntu /bin/bash

# 새탭 열고 
$ docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
6fe2d187b51f        ubuntu1             0.00%               1.996MiB / 1.914GiB   0.10%               3.16kB / 0B         1.24MB / 0B         1

# ubuntu 실시간 정보를 다줌
# top :

$ docker top == ps -aux
```

```bash
# 이미지에 대한 정보
$ docker history --help

# 컨테이너에 대한 히스토리
$ docker logs --help

# 명령어 확인
https://docs.docker.com/engine/reference/commandline/docker/

# 자료 저장 / 공유

$ docker cp

호스트 > 컨테이너 : master@ubuntu:~$ docker cp ./data/test2.txt ubuntu1:/data/
컨테이너 > 호스트 : master@ubuntu:~$ docker cp ubuntu1:/data/test1.txt ./data/
```

### 마운트 

디렉토리를 카피가 아니고 호스트와 연결 -> 후에 컨테이너 간의 데이터 연결

https://docs.docker.com/storage/volumes/

```bash
$ docker run -it --name ubuntu2  -v /data ubuntu /bin/bash

"Mounts": [
            {
                "Type": "volume",
                "Name": "c1bcf701548918eed7e109a3f1cd6151205b278bf1872aaa6ea4b1edaabd3bc7",
                "Source": "/var/lib/docker/volumes/c1bcf701548918eed7e109a3f1cd6151205b278bf1872aaa6ea4b1edaabd3bc7/_data",
                "Destination": "/data",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ]
       
$ docker inspect -f '{{.Config.Volumes}}' ubuntu2
$ docker inspect -f '{{range .Mounts}}{{.Source}}{{end}}' ubuntu2
/var/lib/docker/volumes/c1bcf701548918eed7e109a3f1cd6151205b278bf1872aaa6ea4b1edaabd3bc7/_data

$ sudo ls -l /var/lib/docker/volumes/c1bcf701548918eed7e109a3f1cd6151205b278bf1872aaa6ea4b1edaabd3bc7/_data
total 4
-rw-r--r-- 1 root root 15 Sep 17 04:10 test1.text

# 컨테이너 밖에서 마운트로 엮인 폴더에 들어가서 생성
master@ubuntu:~/data$ sudo touch /var/lib/docker/volumes/c1bcf701548918eed7e109a3f1cd6151205b278bf1872aaa6ea4b1edaabd3bc7/_data/test2.txt

# 컨테이너 안에 들어가보면 들어와있다.
root@31f71c209e50:/data# ls
test1.text  test2.txt

# 이제 우리가 원하는 폴더로 마운트 해보자

$ pwd #로 절대경로를 얻고
$ docker run -it -v /home/master/data:/data --name ubuntu21 ubuntu /bin/bash

$ docker inspect -f '{{range .Mounts}}{{.Source}}{{end}}' ubuntu2
/home/master/data

master@ubuntu:~/data$ docker run -it -v /home/master/data:/data --name ubuntu22 ubuntu /bin/bash
# 할시 같은 폴더를 공유함. NAS 처럼사용가능

```



```bash
$ docker run -it --name ubuntu11 -v data ubuntu /bin/bash

$ docker run -it --name ubuntu12 --volumes-from ubuntu11 ubuntu  /bin/bash

$ docker inspect -f '{{range .Mounts}}{{.Source}}{{end}}' ubuntu11
$ docker inspect -f '{{range .Mounts}}{{.Source}}{{end}}' ubuntu12
가 같음
```



```bash
master@ubuntu:~$ docker volume --help
~$ docker volume ls
~$ docker volume create --name data
master@ubuntu:/var/lib/docker$ docker volume inspect data
[
    {
        "CreatedAt": "2020-09-17T05:14:58-07:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/data/_data",
        "Name": "data",
        "Options": {},
        "Scope": "local"
    }
]

master@ubuntu:/var/lib/docker$ docker run -it -v data:/data --name ubuntu3 ubuntu /bin/bash
root@2438301defc6:/# echo "Hello Docker 1" > ./data/test1.txt

master@ubuntu:~$ sudo ls /var/lib/docker/volumes/data/_data
test1.txt


```

```bash
master@ubuntu:~$ mkdir website
master@ubuntu:~$ mkdir ./website/ROOT
master@ubuntu:~$ mkdir ./website/ROOT/WEB-INF
master@ubuntu:~$ touch ./website/ROOT/index.jsp

# index.jsp 를 꾸미고
master@ubuntu:~$ docker run -it -p 8888:8080 -v /home/master/website/ROOT/:/usr/local/tomcat/webapps/ROOT --name tomcat9 tomcat



```

```bash
# 도커 경로변경

master@ubuntu:~/website/ROOT$ sudo systemctl stop docker
master@ubuntu:~/website/ROOT$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Thu 2020-09-17 05:48:07 PDT; 5s ago

# ~/data 다 비워주고
master@ubuntu:~$ sudo vim /etc/docker/daemon.json
{
	"data-root" : "/home/master/data/docker"
}

$ sudo systemctl start docker
```

