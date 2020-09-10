# Day2



도커의 개념을 잡을때 시작하는 부분은 개발팀의 개발환경 구성하는 방법!



1. 시스템 구성 
   - 통합
   - 분산
2. 서비스 구성
   - 테스트
   - 빌드
   - 배포

### 방법론(개발 문화)

1. On-premise : 인프라 아키텍쳐 중심 - 내부적으로 시스템 구성
2. Off-premise : cloud 중심 - 클라두으 시스템에다가 구성
   - 보안문제 발생.

1. Monolithic

2. Microservice 

   - https://aws.amazon.com/ko/microservices/

   - 서비스 별로 관리팀이 붙어 분화되어 작업함. 관련기능이 많으면 조직도 마이크로화됨

   - 샘플사이트

     1) https://blog.dreamfactory.com/microservices-examples/

     2) https://woowabros.github.io./experience/2018/03/13/k8s-test.html

     3) https://woowabros.github.io/r&d/2017/06/13/apigateway.html

     4) https://engineering.linecorp.com/ko/blog/building-large-kubernetes-cluster/

   * 용어설명

     1) 쿠버네이트 : 도커를 관리하기 위한 기술

     ​	Google 에서 제공한 툴이 쿠버네이트 / 혹은 프로그램 언어로 직접 관리 가능

     2)  API gateway

     ​	클라이어늩 앱과 마이크로서비스 사이에 위치하여 운영을 쉽게 할 수 있게끔

     3) 도커

     ​	마이크로 서비스를 위한 구성

     ​	-> LXC(리눅스 컨테이너) + 마이크로 서비스

     ​	도커란걸 사용해서 이미지의 복사본을 만들어 컨테이너를 구성

     ​	이미지 : 클래스 컨테이너 : 인스턴스 같이

     ​	레파지토리 : 서비스에 대한 이미지가 저장됨 -> 다운로드 -> 컨테이너 -> 서비스

     1. ​	
         1. 저장된 이미지를 사용하는 방법 < 보통 이거중심으로 얘기
         2. 사용자 정의 이미지를 저장하는 방법 

     도커허브에서 이미지를 불러온다 컨테이너를 만든다 사용한다.

     도커는 클라이언트와 호스트로 나뉘어짐. 

     ```bash
     # 클라이언트
     $ docker run ~~~
     
     # 호스트 
     docker damon (데몬, 눈에 안보인다)
     
     # 
     $ sudo systemctl status docker
     
     # 도커의 시스템 상태 파악
     $ sudo docker info
     ```

     도커허브에서 이미지를 가져옴

     https://hub.docker.com

     ```bash
     master@ubuntu:~/docker$ sudo docker run --help
     # 최신버젼 가져오기
     master@ubuntu:~/docker$ sudo docker run 옵션 이미지(이미지:latest)
     
     # 실행
     $ sudo docker run ubuntu:latest
     
     # 상태
     $ sudo docker container --help
     $ sudo docker container ls / ps 컨테이너 리스트 출력 (-a 실행중지된것도출력)
     
     # 이미지 확인 (원본파일들)
     $ sudo docker images
     
     # sudo 없이 docker 사용하기
     $ sudo usermod -aG docker $USER #현재 유저에게 도커는 sudo권한이있어!
     $ reboot
     
     # 이미지 이름변환
     $ docker run --name ubuntu1 ubuntu:lastest
     
     # 이미지와 상호작용하기
     $ docker run -it --name ubuntu4 ubuntu:latest
     -> OS가 설치된 내용을 다운로드 받아서 부팅한 개념이 아님
     -> OS는 우리가 사용하는 ubuntu이고 이 위에서 bash 만을 실행시킨것
     
     상호작용중에 새 터미널 열고 ps 보면 해당 이미지 실행중
     exit시 다시 exit상태됨
     
     
     $ docker start --help
     하나 혹은 이상의 컨테이너를 실행시킨다
     
     $ docker start ubuntu2
     >> ubuntu2 
     
     $ docker start -i ubuntu2 # 상호작용함
     >> Hello Docker
     
     둘다 실행했다가 끝남
     
     $ docker attach --help
     
     # 입출력
     $ docker attach ubuntu4
     
     # running container 에서 커맨드를 실행시킴
     $ docker exec --help
     
     $ docker logs --help
     
     #h 컨테이너 지우기
     $ docker rm --help
     
     # 이미지 지우기
     $ docker rmi ubuntu:latest 
     
     ```

     도커 컨테이너는 다양한 OS를 제공

     - ubuntu
     - centos

     ```bash
     $ docker run -it --name ubuntu1 ubuntu:latest
     $ docker run -it --name centos1 centos:latest
     
     $ uname -na 실행시 두가지 모두 똑같이 나옴(배경 os가 같으니) 
     
     # jdk를 깔아보자 ubuntu1 열려있는상태에서
     /# apt-get update
     /# apt-get install openjdk-8-jdk
     
     # 그냥 vi 쓰면 에러 날수도있어 vim 설치 자바하나짜서 실행해보자
     /# apt-get install vim
     ```

     자바는 

     ​	jre : 실행환경

     ​	jdk : 개발환경

     ```bash
     $ docker run --name jdk8 -it openjdk:8-jdk-slim
     #jdk 설치함 javac version 확인가능
     
     $ docker run --name jre8 -it openjdk:8-jre-slim
     #jre 설치함 java version 확인가능, javac 불가
     
     $ docker exec ubuntu1 java HelloWorld
     # 프로그램을 돌릴수있음
     ```

     