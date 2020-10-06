# day7



```bash
docker ps / images / volume 을 확인하여
전부 초기화하기
mkdir docker
cd docker
mkdir tomcat
cd tomcat
pwd

```

```dockerfile
$ vi Dockerfile

[Docker file 파일 편집]
FROM openjdk:11-jre-slim
MAINTAINER Foo Bar <foo@bar.com>

RUN apt-get update
RUN apt-get -y install wget

WORKDIR /usr/local/
RUN wget [브라우저에서 tomcat.apache.org로 접속하여 tar.gz 링크를 복사 붙여넣기]
RUN tar xvzf apache-tomcat-9.0.38.tar.gz
RUN rm -f apache-tomcat-9.0.38.tar.gz

EXPOSE 8080

CMD /usr/local/apache-tomcat-9.0.38/bin/catalina.sh run

$ docker build -t mytomcat:v0.1 .

docker images
docker run -it -d -p 8080:8080 --name tomcat1 mytomcat:v0.1
curl 127.0.0.1:8080


```

도커 레포지토리 만들고

우분투에는 무려 git 이깔려있다



```bash
master@ubuntu:~$ git version
git version 2.25.1
master@ubuntu:~$ git clone https://github.com/unsung107/Dockerfiles
Cloning into 'Dockerfiles'...
warning: You appear to have cloned an empty repository.
master@ubuntu:~$ ls
data   Desktop  Dockerfiles  Downloads  jdbc.jsp                       Music     Public     Videos
datas  docker   Documents    index.jsp  mariadb-java-client-2.6.2.jar  Pictures  Templates  website
master@ubuntu:~$ cd Dockerfiles/


master@ubuntu:~/Dockerfiles$ mv ../docker/tomcat/ .
master@ubuntu:~/Dockerfiles$ ls
tomcat
master@ubuntu:~/Dockerfiles$ git add .
master@ubuntu:~/Dockerfiles$ git config user.email "unsung102@naver.com"
master@ubuntu:~/Dockerfiles$ git config user.name "unsung107"

```

이제 docker hub 에 Repository 만들어보자( korea.ac.kr )

docker hub Repository 의 이름은 이미지 이름과 같이 하는게 가장 좋다

```bash
master@ubuntu:~/Dockerfiles$ ls
tomcat
master@ubuntu:~/Dockerfiles$ mkdir ubuntu
master@ubuntu:~/Dockerfiles$ cd ubuntu
master@ubuntu:~/Dockerfiles/ubuntu$ ls
master@ubuntu:~/Dockerfiles/ubuntu$ vim Dockerfile

[Dockerfile 편집]
FROM ubuntu:latest
MAINTAINER 자신의id <자신의id@test.com>

RUN apt-get update
RUN apt-get -y install vim

[파일 편집 종료 :wq]

docker build -t myubuntu:v0.1 .

docker run -it --name myubuntu myubuntu:v0.1
```

```bash
$ docker login --help
$ docker logout --help
$ docker tag --help # 

master@ubuntu:~/Dockerfiles/ubuntu$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: unsung107
Password: 
WARNING! Your password will be stored unencrypted in /home/master/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
master@ubuntu:~/Dockerfiles/ubuntu$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
myubuntu            v0.1                f490298e95e3        6 minutes ago       165MB
mytomcat            v0.1                fe862b621d37        59 minutes ago      251MB
ubuntu              latest              9140108b62dc        10 days ago         72.9MB
openjdk             11-jre-slim         a982abd9eead        3 weeks ago         204MB
master@ubuntu:~/Dockerfiles/ubuntu$ docker tag myubuntu:v0.1 unsung107/myubuntu:v0.1
master@ubuntu:~/Dockerfiles/ubuntu$ docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
myubuntu             v0.1                f490298e95e3        7 minutes ago       165MB
unsung107/myubuntu   v0.1                f490298e95e3        7 minutes ago       165MB
mytomcat             v0.1                fe862b621d37        About an hour ago   251MB
ubuntu               latest              9140108b62dc        10 days ago         72.9MB
openjdk              11-jre-slim         a982abd9eead        3 weeks ago         204MB

master@ubuntu:~/Dockerfiles/ubuntu$ docker push unsung107/myubuntu:v0.1

```



우선 myubuntu 와 관련된 container 삭제 후 태그삭제 후 이미지 삭제

```bash
master@ubuntu:~/Dockerfiles/ubuntu$ docker pull unsung107/myubuntu:v0.1
master@ubuntu:~/Dockerfiles/ubuntu$ docker pull neorati/myubuntu:v0.1

```



지금까진 퍼블릭 레지스트리임



private 은 docs.docker.com/registry 에서 확인가능

프라이빗 레지스트리를 써보자

```bash
master@ubuntu:~/Dockerfiles/ubuntu$ docker pull registry
master@ubuntu:~/Dockerfiles/ubuntu$ docker run -it -d -p 5000:5000 --name docker-registry registry
master@ubuntu:~/Dockerfiles/ubuntu$ docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete 
Digest: sha256:4cf9c47f86df71d48364001ede3a4fcd85ae80ce02ebad74156906caff5378bc
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
master@ubuntu:~/Dockerfiles/ubuntu$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
registry            latest              2d4f4b5309b1        3 months ago        26.2MB
hello-world         latest              bf756fb1ae65        9 months ago        13.3kB
master@ubuntu:~/Dockerfiles/ubuntu$ docker tag hello-world localhost:5000/hello-world
master@ubuntu:~/Dockerfiles/ubuntu$ docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
registry                     latest              2d4f4b5309b1        3 months ago        26.2MB
hello-world                  latest              bf756fb1ae65        9 months ago        13.3kB
localhost:5000/hello-world   latest              bf756fb1ae65        9 months ago        13.3kB
master@ubuntu:~/Dockerfiles/ubuntu$ docker push localhost:5000/hello-world
The push refers to repository [localhost:5000/hello-world]
9c27e219663c: Pushed 
latest: digest: sha256:90659bf80b44ce6be8234e6ff90a1ac34acbeb826903b02cfa0da11c82cbc042 size: 525
master@ubuntu:~/Dockerfiles/ubuntu$ curl -X GET http://localhost:5000/v2/_catalog
{"repositories":["hello-world"]}
master@ubuntu:~/Dockerfiles/ubuntu$ curl -X GET http://localhost:5000/v2/hello-world/tags/list
{"name":"hello-world","tags":["latest"]}
master@ubuntu:~/Dockerfiles/ubuntu$ docker exec -it docker-registry ls /var/lib/registry/docker
registry
failed to resize tty, using default size


```



이번엔 nginx 를해보자



```bash
master@ubuntu:~/Dockerfiles$ mkdir nginx
master@ubuntu:~/Dockerfiles$ cd nginx/
master@ubuntu:~/Dockerfiles/nginx$ ls
master@ubuntu:~/Dockerfiles/nginx$ vim Dockerfile
master@ubuntu:~/Dockerfiles/nginx$ 

FROM ubuntu:latest
MAINTAINER unsung107 <unsung107@korea.ac.kr>

RUN apt-get update
RUN apt-get -y install nginx

WORKDIR /etc/nginx

EXPOSE 80

CMD [ "nginx", "-g", "daemon off;" ]

master@ubuntu:~/Dockerfiles/nginx$ docker build -t registry-nginx .
master@ubuntu:~/Dockerfiles/nginx$ docker tag registry-nginx localhost:5000/registry-nginx
master@ubuntu:~/Dockerfiles/nginx$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
registry-nginx                  latest              0d130def767b        2 minutes ago       156MB
localhost:5000/registry-nginx   latest              0d130def767b        2 minutes ago       156MB
ubuntu                          latest              9140108b62dc        10 days ago         72.9MB
registry                        latest              2d4f4b5309b1        3 months ago        26.2MB
hello-world                     latest              bf756fb1ae65        9 months ago        13.3kB
localhost:5000/hello-world      latest              bf756fb1ae65        9 months ago        13.3kB
master@ubuntu:~/Dockerfiles/nginx$ docker image push localhost:5000/registry-nginx
The push refers to repository [localhost:5000/registry-nginx]
dab0796ed04a: Pushed 
541b54a92ec4: Pushed 
782f5f011dda: Pushed 
90ac32a0d9ab: Pushed 
d42a4fdf4b2a: Pushed 
latest: digest: sha256:f2468605d18dcdb8ad396022b996ae6589b46abefeebd31fe687664073b8fb06 size: 1367
master@ubuntu:~/Dockerfiles/nginx$ curl -X GET http://localhost:5000/v2/_catalog
{"repositories":["hello-world","registry-nginx"]}

```



확인을 위해 이미지를 다지워주자 (registry 만 제외하고)



```bash
master@ubuntu:~/Dockerfiles/nginx$ docker image pull localhost:5000/registry-nginx
master@ubuntu:~/Dockerfiles/nginx$ docker run -it -d -p 80:80 --name nginx1 localhost:5000/registry-nginx
0509b9ba97262b751b9464ab1686a9b149bbd6a0e34ff58a705217984ee0b4cb
master@ubuntu:~/Dockerfiles/nginx$ curl 127.0.0.1:80

```

registry 없어지면 저장 이미지 데이터도 삭제됨.

volume 처리도하자

​	registry container : /var/lib/registry/docker/registry/v2

​	외부 host : /home/master/registry_data

```bash
$ docker run -it -d -p 5000:5000 -v /home/master/registry_data:/var/lib/registry/docker/registry/v2 --name docker-registry registry
```

