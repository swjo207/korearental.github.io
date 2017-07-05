---
layout: post
title: "데이터베이스 클러스터 설치& 설정 가이드"
description: "DB cluster installation & Settings"
date: 2017-01-19
tags: [database, clutser, install, setting]
comments: true
share: true
---

목 차
=====

[*목 차* ](#목-차)

[*1.* *개요* ](#개요)

[*1.1* *용어 정의* ](#용어-정의)

[*1.2* *제약 사항* ](#제약-사항)

[*2.* *준비물* ](#준비물)

[*2.1* *환경 설정* ](#환경-설정)

[*2.2* *우분투 설치 USB* ](#우분투-설치-usb)

[*2.3* *서버 설치 순서* ](#서버-설치-순서)

[*3.* *우분투 설치* ](#우분투-설치)

[*3.1* *우분투 설치* ](#우분투-설치-1)

[*4.* *DB 설치* ](#db-설치)

[*4.1* *DB 설치* ](#db-설치-1)

[*4.2* *DB Cluster 설정* ](#db-cluster-설정)

[*4.3* *Grand Access for HAProxy Server* ](#grand-access-for-haproxy-server)

[*4.4* *HA Proxy 설치* ](#ha-proxy-설치)

[*4.5* *설치 검증하기* ](#설치-검증하기)

[*5.* *Q & A* ](#q-a)

[*5.1* *DB 접속 오류* ](#db-접속-오류)


## 1.  개요

### 1.1  용어 정의

-   DESS (Diskless Enterprise System Suite) Basic Edition 의 설치에 앞서 용어의 혼선을 피하기 위하여 미리 용어를 정의하여 후술하는 설치 가이드 문서는 이를 따르기로 한다.

-   DESS: Diskless Enterprise System Suite 의 약자로서, Diskless 시스템의 공식 명칭으로서 후술하는 본 시스템을 줄여서 부르기로 함

-   노드: DESS 시스템 세트 한 개당 한 개의 노드를 구성한다. IT 인프라 수요처가 여러 개의 노드로 구성하여 도입하는 경우 DESS 시스템과 혼동을 피하기 위하여 DESS 개별 독립 망을 구분하여 칭하기로 함

-   MMS: Main Management Server 의 약자로서, DESS 시스템 내부를 구성하는 다수 서버의 일종이다. Basic 에디션에서는 구분하지 않으며, 스탠다드 이상 에디션에서 사용된다. 주로 스토리지 관리를 담당하는 서버 요소 임

-   BMS: Boot Management Server 의 약자로서, DESS 시스템 내부를 구성하는 다수 서버의 일종이다. Basic 에디션에서는 구분하지 않으며, 스탠다드 이상 에디션에서 사용된다. 주로 클라이언트 PC의 부팅 제어를 관리하는 서버 요소 임

-   DBMS: 디비 서버로 통칭하여 부르기도 함. 지속적인 시스템 데이터를 저장하고 관리하는 서버 요소로서, 스탠다드 이상 에디션에서 따로 서버 하드웨어로 분리하여 사용함

-   CMS: Central Management Server 의 약자로서, 선택적으로 설치가 가능한 서버 요소이다. 여러 개의 노드를 관리하기 위한 목적으로 사용된다.

-   클라이언트: 1개의 노드에 접속하는 개별 PC를 서버와 구분하여 클라이언트라고 부르기로 한다. PC라고 할 때, 하드웨어적인 의미가 있기 때문에 혼동이 발생할 수 있어 “클라이언트 PC” 또는 클라이언트라고 통칭하기로 한다.

-   에이전트: Agent, 클라이언트에 설치되는 유틸리티 소프트웨어로서 주로 클라이언트의 상태 (네트워크 연결 여부) 를 체크하기 위한 용도로 초기 OS 설치 시 함께 깔리게 된다.

### 1.2  제약 사항

-   DB Cluster를 구성하기 위해서는 홀수 개 및 3대 이상 구성해야 한다.

-   2대로 구성하는 경우, 장애 및 복원 과정에서 실패할 수 있다.

-   DESS Ultimate Edition 에서 사용하는 것을 가정하고 있지만, Professional Edition 에서도 업그레이드 할 수 있으며 프로 에디션과 중요한 차이점이기도 하다.

## 2.  준비물

### 2.1   환경 설정

-   로컬서버 설치를 수행함에 앞서 적합한 네트워크 환경을 조성해야 한다.

-   DB 서버를 3대를 기준으로 설치를 하는 것을 가정한다.

-   서버 H/W 는 2G 이상 CPU, 4G 메모리 이상, 128G 이상의 디스크를 준비한다.

### 2.2 우분투 설치 USB

-   전달 받는 iso 파일은 부팅가능한 usb로 만든다.

-   USB 종류에 따라 이미지 굽는 것이 실패할 수도 있으니 Sandisk 를 추천하며, 이미지를 USB로 굽는 소프트웨어는 unetbootin 소프트웨어를 사용하는 것을 추천한다. ([*https://unetbootin.github.io/*](https://unetbootin.github.io/))

-   또는 다음 사이트를 참고 ([*https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-windows*](https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-windows))

### 2.3 서버 설치 순서

1.  서버 운영체제 (Ubuntu 14.04) 설치

    1)  운영체제 및 패키지 설치

    2)  네트워크 설정

2.  데이터베이스(DBMS) 설치

    1)  콘솔 사용하여 자동 설치 됨

3.  Proxy 설치 및 클러스터 구성

    1)  콘솔 사용하여 자동 설치 됨

## 3. 우분투 설치

-   <span id="_Toc479343209" class="anchor"></span>서버 설치는 리눅스 운영체제 위에 동작하도록 설계 되어 있어, 이 글을 읽고 있는 엔지니어는 리눅스 운영체제에 대해 어느 정도 지식을 가지고 있는 것을 가정하기로 한다. (이에 대해서는 다음 사이트를 참고함 - https://tutorials.ubuntu.com/tutorial/tutorial-install-ubuntu-desktop)

-   또한 리눅스 사용자 가이드([*https://help.ubuntu.com/stable/ubuntu-help/*](https://help.ubuntu.com/stable/ubuntu-help/))를 참고하여 서버 관리에 활용할 수 있도록 추천함

-   DB 서버 3대에 각각 우분투를 설치하는 과정을 거친다.

-   각 DB 서버에 할당한 IP를 미리 정하여, 순차적으로 할당하도록 한다.

-   이하 설치 과정에서 H/W 디스크 구성에 따라 용어가 달라질 수 있으므로, 상황에 따라 변경하여 설치를 진행해야 한다.

### 3.1 우분투 운영체제 설치

#### 1)  부팅 USB를 사용하여 우분투 설치 환경으로 부팅한다.

#### 2)  BIOS 설정에 부팅 순서가 USB 부팅이 1순위로 되어야 한다. (HDD나 CD 등이 1순위면 부팅이 정상적으로 되지 않는다.)

#### 3)  Install Ubuntu를 선택한다. (실제 디스크에 설치하는 과정)

<img src="./db_cluster_install/media/image1.png" width="412" height="302" />

#### 4)  와이파이 모양 아이콘 -&gt; Edit Connections를 누른다. (Ethernet 리스트는 하드웨어 스펙에 따라 상이함)

<img src="./db_cluster_install/media/image2.png" width="404" height="302" />

#### 5)  Wired connection 1을 선택하고 Edit 버튼을 누른다.

#### 6)  Editing Wired connection 1 창에서 IPv4 Settings 탭으로 이동한다.

<img src="./db_cluster_install/media/image3.png" width="402" height="302" />

#### 7)  Method를 Manual로 변경하고 Add버튼을 눌러 Address, Network, Gateway, DNS servers를 입력한다. (아이피 정보는 네트워크 환경에 따라 상이함)

<img src="./db_cluster_install/media/image4.png" width="403" height="302" />


#### 8)  저장 버튼을 눌러 인터넷에 연결되는지 체크한다.

#### 9)  인터넷이 연결되었다면 Continue 버튼을 누른다.

<img src="./db_cluster_install/media/image5.png" width="350" height="262" />

#### 10)  Something else를 누르고 Continue 버튼을 누른다.

<img src="./db_cluster_install/media/image6.png" width="344" height="258" />


#### 11)  우분투를 설치 할 /dev/sda1(100G)를 선택하고 Change를 누른다.

#### 12)  Edit partition 창에서 Use as는 ext4, Mount point는 /로 한다.

#### 13)  /dev/sda1을 제외한 나머지 파티션은 기억한다. (/dev/sdb와 같이 숫자로 끝나지 않는 것은 뺌)

<img src="./db_cluster_install/media/image7.png" width="384" height="287" />

#### 14)  /dev/sda1을 선택하고 인스톨을 시작한다.

#### 15)  이어서 나오는 Do you want to return to the partitioning menu? / Do you want to return to the partitioner? 에서는 모두 Continue를 선택한다.

<img src="./db_cluster_install/media/image8.png" width="403" height="302" />

<img src="./db_cluster_install/media/image9.png" width="405" height="302" />

#### 16)  이어서 나오는 지역 설정, 키보드 설정은 default 값으로 한다.

#### 17)  Who are you? 에서는 사용자 계정을 만든다.

#### 18)  사용자 계정은 우분투 로그인할 때 사용되니 반드시 기억한다.

<img src="./db_cluster_install/media/image10.png" width="468" height="349" />

#### 19)  재 부팅 후 BIOS 설정에서 부팅 순서를 HDD 부팅이 1순위로 변경한다.



## 4.  DB 설치

### 4.1  DB 설치


1)  3대의 DB서버에 각각 원격 접속한다.

2)  개별 서버 마다 다음 과정을 반복 수행한다.

3)  터미널을 열고, root로 로그인 한다.

4)  아래 명령어를 실행하여 설치 스크립트 (Installer.py) 실행 후 순서에 따른다.

5)  \# python /Diskless\_Installer/installer.py

6)  \[1\] Install Database 를 선택한다.

7)  이어서 \[1\] Install Database를 선택한다.

8)  실행하면, 하단의 그림과 같이 설치가 자동으로 진행이 되면서 설치가 완료된다.

<img src="./db_cluster_install/media/image11.png" width="554" height="351" />

### 4.2 DB Cluster 설정

1)  앞의 과정이 성공한 경우 다음을 실행한다.

2)  클러스터 대상이 되는 IP를 전부 입력한다.

3)  E.g.) 첫번째 붉은색 입력 (Cluster IP Address = e.g. 192.168.0.1, ~~~)

4)  <img src="./db_cluster_install/media/image12.png" width="516" height="255" />다음 물음이 나오면 현재 서버 IP를 입력한다. (Current Cluster IP Address)

5)  다른 서버에도 위 과정을 동일하게 진행한다.

\*\* 다음 명령어는 한 개 서버에서 설정 하면 되고, 모든 컴퓨터에서 설정 하지 않아도 된다.

6)  입력 완료 되면 서비스 재 시작 하여야 한다. (단 1개의 서버에서만 진행 하면 된다. 그 외의 서버는 재 시작 하면 된다.)

<img src="./db_cluster_install/media/image13.png" width="642" height="103" />

7)  서비스 재 시작 후 HA Proxy 에 대한 설정을 진행하게 된다.

### 4.3 Grand Access for HAProxy Server


1) 3개 DB 서버마다 다음을 실행한다.

2)  \# python /Diskless\_Installer/installer.py 명령을 실행

3)  \[3\]. Install HAProxy 을 선택

4)  \[2\]. Grant Permission For HAProxy User 을 선택

5)  <img src="./db_cluster_install/media/image14.png" width="516" height="256" />HA Proxy 서버 IP를 입력하고 DB 암호를 입력한다.

### 4.4 HA Proxy 설치


1)  DB 서버에 접속하는 모든 서버에서 다음 과정을 수행한다.

2)  \# python /Diskless\_Installer/installer.py 실행한다.

3)  \[3\]. Install HAProxy 을 선택한다.

4)  \[1\]. Install HAProxy Load Balance 을 선택한다.

5)  다음 화면처럼 출력되고,

6)  <img src="./db_cluster_install/media/image15.png" width="597" height="209" />프록시 서버 IP를 입력하고, DB 서버 수를 입력한다.

7)  다음으로 각 데이터베이스 IP 를 입력한다.

8)  <img src="./db_cluster_install/media/image16.png" width="587" height="270" />정상적인 경우 다음과 같은 화면이 출력된다.

### 4.5 설치 정상여부 검증하기

1)  임의의 DB 서버에 접속하여 다음과 같이 화면이 출력되면 성공한 것이다.

<img src="./db_cluster_install/media/image17.png" width="564" height="243" />

## 5. Q & A

### 5.1 DB 접속 오류 시 

1)  4.3 과정을 다시 반복한다.
