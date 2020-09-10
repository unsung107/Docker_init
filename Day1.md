# Day 1

## Docker Program

1. Hardware
2. OS(Platform)
3. Networking
4. Application

기존에는 워낙 딱딱하게 고정되어있어 옮기거나 수정을 원하면 복잡한 마이그레이션이 필요했음.

기존 => Immutable 한 서버구조 : 변경이 불가능함

물리서버가 좋아짐에따라 운영체제를 하나가 아닌 여러개를 올릴수가 있어짐

mutable한 서버구조 -> 가상화 -> 하드웨어적으로 가상화를 지원해줘야함(AMD계열의 CPU는 지원하지 않음. Intel + board 에서 지원.(예전(i5이전)에는 cmos딴에서 막아둬서 해제 vt-x / virtualiztion 를 해야했음 ) )

반 가상화(그중 하나가 docker) 일부분만 가상화를 시킨구조.



가상화 소프트웨어

1. vmware by vmware
2. virtual box by oracle



careate a new vm

다운받은 iso t너택

ubuntu 선택

경로를 C: 바로 밑에 

c:\Virtual Machines\Ubuntu 64-bit 에설치

디스크용량을 크게(바로 잡진않고 점차 크게함)

화면이 나오면 한국어 우분투설치 계속하기 최소설치 계속하기 지금설치 계속하기



ubuntu termial 실행후

sudo  

sudo apt-get update 암호입력

sudo apt-get upgrade

---

설치후 설치 폴더에서 폴더 복사하면 운영체제 복사임.

하나 하나의 폴더가 하나하나의 운영체제



*리눅스

chroot - 가상디렉토리구조

LXC - linux container

​				LXD

​				LXC

​				openVZ

​				Virtuozzo

​				docker 란 프로그램들이 만들어짐

window 에 도커를 생성하게되면 리눅스 기반으로만들어졌기때문에 hiper V라는 가상화를 추가시켜줘야함.

---

도커는



도커 클라이언트(cli 모드)

도커 서버로 이루어짐(deamon - status)

```bash
$ sudo systemctl status docker
$ sudo docker -version
```

