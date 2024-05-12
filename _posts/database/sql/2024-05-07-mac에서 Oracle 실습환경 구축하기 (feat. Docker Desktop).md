---
layout: post
title: mac에서 Oracle 실습환경 구축하기 (feat. Docker Desktop)
categories: 
  - database
  - sql
sitemap: false
hide_last_modified: true
related_posts:
---
![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=http%3A%2F%2Fhits.dwyl.comjmj-gif%2FJMJ-GIF.github.io.svg{{ page.url }}&count_bg=%2379C83D&title_bg=%23555555&icon=jenkins.svg&icon_color=%23E7E7E7&title=%EC%A1%B0%ED%9A%8C%EC%88%98&edge_flat=true)

## 들어가며
SQL을 공부하기 위해 이것저것 찾아보면, 요새 괜찮은 자료들이 참 많은 것 같습니다. 유/무료 강의들도 잘나와 있고, 관련한 서적들도 많이 출판된 것 같습니다. 하나의 자료들을 정해서 쭉 따라가다 보면 어떻게 쿼리해야하는지에 대한 설명들이 많이 나와있는데, 그럴때마다 실제로 쿼리를 날려보면 괜찮겠다라는 생각이 많이 들었습니다. 
저는 SQLD를 준비하면서 SQL을 처음 접했었는데요, SQLD를 공부해본 분들은 아시겠지만 실기 문제가 없기 때문에 사실상 SQL과 관련된 책을 읽고 상상(?) 코딩을 해야하는 경우가 많습니다. SQLD는 난이도가 쉬운 편이라 어찌어찌 취득했지만, SQLP를 준비하면서 실제로 쿼리를 날려보고 실험해보는 것에 대한 필요를 강하게 느꼈습니다.

실제 서비스에 사용되는 DB를 구축하는 것은 고려해야할 부분이 상당히 많지만 (비용, 속도, 동시성 등), 개발을 위해 나만의 DB를 만드는 것은 그렇게 어렵지 않습니다! 같이 차근차근 따라해보면 어느덧 나만의 DB가 컴퓨터에 깔려있을거에요.

## 로컬 실습 환경
* Mac OS Sonoma 14.3
* OS/Arch : darwin/arm64
* Apple M2
* Docker version 24.0.2
* Docker Desktop 4.21.1

## 배경 지식
이 글을 이해하기 위해서, 아래와 같은 배경지식이 있다고 가정합니다.
* docker 사용방법

## Docker를 사용해야 하는 이유
우선 docker가 무엇이고, docker를 어떻게 설치해야하는지에 대해서는 이 글의 주제에 벗어난 것이기 때문에 따로 설명하지는 않겠습니다! 
관련해서 이미 잘 설명된 수많은 포스트가 있기 때문에 쉽게 익히실 수 있을거에요! 그런데 왜 굳이 Oracle을 설치하기 위해 docker를 사용해야 할까요?

1. Oracle은 Mac에서 직접적으로 돌아가도록 지원하지 않음
   * Oracle은 Customization이 까다로워 개발에 있어 어려움이 있는 것에 비해 그만큼의 수익이 나지 않는 것을 이유로, Mac에서 설치가능한 버전을 제공하지 않습니다. → <a href="https://www.quora.com/Why-doesnt-Oracle-support-Mac-OS">링크</a>
   * 그렇기 때문에 Oracle을 mac에 설치하기 위해서는, Virtual Machine을 사용하거나 docker를 사용해서 독립된 환경을 구성해주는 것이 필수적입니다.

     
2. 다양한 버전을 빠른 속도로 설치/삭제 할 수 있음
   * SQL과 관련한 여러 강의를 듣다보면, 자료별로 Oracle 버전이 다르거나 심지어는 Mysql, Postgre 같이 DB엔진 자체가 다른 경우가 많이 있을 것같습니다. 심지어는 같은 Oracle인데 다른 버전을 바꿔가며 쿼리해야하는 경우도 존재할 수 있습니다. 이런 경우 버전 전환이나 설치/삭제가 상대적으로 docker가 매우 빠르기 때문에, docker를 사용하는 것이 큰 이점이 될 수 있습니다.
   * 이때, 주의해야할 점은 CPU 아키텍처에 맞는 이미지를 사용해야 한다는 것입니다. 도커는 VM과 비슷하게 실행환경을 격리해주지만, 도커는 host platform과 OS를 공유하기 때문에 아키텍처가 다른 이미지는 실행이 되지 않거나 의도하지 않는 오작동을 일으킬 수 있습니다. 때문에 현재 실습에서는 Arm64에 맞는 이미지를 찾아 설치하도록 하겠습니다.

## Docker Desktop 으로 Arm64 기반 Oracle 설치하기
저는 Docker Desktop 4.21.1 버전을 사용하고 있습니다.

docker daemon 을 켜고 아래 커맨드를 입력하면 설치된 버전을 확인할 수 있습니다.
~~~bash
docker version
~~~
가장 먼저 어떤 oracle 버전을 설치해야할지 결정해야 할텐데, 학습하고 계시는 강의나 서적이 있으시다면 적혀있는 버전을 찾아 다운로드 받으시면 되고, 어떤 버전이든 상관없으시다면 저와 같은 버전을 다운로드 받으시면 될 것 같습니다.

오라클은 에디션과 버전으로 소프트웨어를 구분하는데요, 둘의 차이점은 아래와 같습니다. (참고 : [링크](https://bangu4.tistory.com/321))
* 에디션
  * 에디션은 오라클 제품군 내에서 기능 및 사용 가능한 옵션을 정의합니다.
  * 주로 사용되는 오라클 데이터베이스 에디션은 Standard Edition(SE), Enterprise Edition(EE), Express Edition(XE) 등이 있습니다.
  * 각 에디션에는 다양한 기능과 한도가 있으며, 비즈니스 요구사항에 맞게 선택됩니다. 예를 들어, Enterprise Edition은 풍부한 기능을 제공하며 큰 규모의 엔터프라이즈 애플리케이션에 적합합니다. 반면에 Standard Edition은 중소 규모의 비즈니스 애플리케이션에 적합하며 더 저렴한 비용으로 제공됩니다.
* 버전
  * 버전은 특정 제품의 릴리스를 식별합니다. 오라클 제품은 주로 숫자와 마침표로 이루어진 버전 번호를 가집니다.
  * 새로운 버전은 보통 이전 버전의 기능을 개선하거나 추가합니다. 또한 보안 패치, 성능 향상 및 새로운 기능이 도입될 수 있습니다.

저는 혼자 개발용도로 사용하고 싶고, 상업적인 목적은 없으므로 에디션은 XE로 선택했습니다. 또한 보통 연습용으로 사람들이 많이 선택하는 버전이 11g인 것을 확인하고, oracle-xe-11g와 관련된 이미지를 찾기로 결정했습니다.

### 첫 번째 시도
가장 먼저 oracle 공식 이미지가 있는지 찾아보았습니다.
  
![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/1.png "Oracle 공식 이미지")

공식 이미지가 있었고, version 9 까지 지원을 하고 있어 우선적으로 간단하게 해당 이미지를 다운받아 보았습니다. 

port는 oracle 기본 포트는 1521, platform은 호스트 환경에 맞추어 linux/arm64/v8로 다운을 받았습니다.

~~~bash
docker run -d -p 1521:1521 --name oracle9 --platform=linux/arm64/v8 -it oraclelinux:9
~~~

이후 만들어진 컨테이너 내로 들어가 sqlplus를 실행해 보았습니다.

(여기서 sqlplus는 oracle db와 사용자가 소통할 수 있게 도와주는 CLI 툴이라고 생각하시면 됩니다. 초기 유저 설정 및 데이터 insert를 위해 sqlplus를 통해 반드시 접속해야 합니다.)

~~~bash
docker exec -it oracle9 sqlplus
~~~

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/2.png "터미널 사진")

그런데 sqlplus Command가 아예 존재하지 않았습니다. `/bin` 경로 내에 sqlplus가 설치되지 않았고, 다른 버전의 이미지들도 마찬가지여서 다른 방법으로 설치를 해야겠다고 생각했습니다.

### 두 번째 시도

[Oracle 공식 github](https://github.com/oracle/docker-images/tree/main)를 찾아보던 중, 수동으로 dockerfile을 만들어 설치하는 방법에 대해 발견할 수 있었습니다. 

[해당 페이지](https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance)에서는 Arm64 기반 아키텍처 관련해서 Oracle 19c 만 제공하고 있어, 19c 이미지로 진행하겠습니다.



* github repo를 clone 합니다.

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/3.png "github 레포사진")    



* [Oracle 홈페이지](https://www.oracle.com/database/technologies/oracle19c-linux-arm64-downloads.html)에서 LINUX._ARM64_1919000_db_home.zip을 다운로드 받습니다. Oracle 계정을 생성해야 합니다.


![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/4.png "Oracle 홈페이지")   



* 다운로드 받은 파일을 아래의 경로에 위치시킵니다.
   * `./docker-images/OracleDatabase/SingleInstance/dockerfiles/19.3.0` 이때 file name은 `LINUX._ARM64_1919000_db_home.zip` 입니다.

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/5.png "19.3.0")   
   


* 상위 경로로 이동하여(`./docker-images/OracleDatabase/SingleInstance/dockerfiles`) 터미널에서 다음 명령어를 실행시킵니다. 설치가 완료되면 docker image를 확인해줍시다.
~~~bash
./buildContainerImage.sh -v 19.3.0 -e
~~~

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/6.png "터미널 사진2")   
   



* 컨테이너를 띄워줍니다.
~~~bash
docker run -d -p 1521:1521 -it --name oracle19 -e ORACLE_SID={your SID} -e ORACLE_PWD=pass {image_number}
~~~

저는 ORACLE_SID=ORCLCDB, ORACLE_PWD=pass 로 설정했습니다.
    

* 컨테이너 내부 진입 (ID : system, PW : pass)
~~~bash
docker exec -it oracle19 sqlplus
~~~

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/7.png "터미널 사진3")

컨테이너를 띄우고 바로 들어가면, 아래와 같은 메시지가 나옵니다. 이는 ORACLE이 아직 initialization을 하고 있다는 의미로, 초기 세팅이 진행되고 있는데 접속을 시도해서 나오는 에러입니다.

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/8.png "터미널 사진4")     

컨테이너 로그로 들어가셔서 구축이 완료되면 그 때 sqlplus로 다시 접속해보세요!

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/9.png "Docker Desktop")    

이후 DATABASE IS READY TO USE! 가 뜨면 완료입니다!



## DBeaver에 연결하기
DBeaver 버전23.0.1  기준

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/10.png "DBEAVER")

Database 에는 설정한 ORACLE_SID 를 입력해줍니다. 저같은 경우, ORACLCDB 로 설정했어서 해당 정보를 입력해주었습니다.

포트는 1521입니다.

username 에는 접속하고자 하는 유저 이름을, password에는 그에 맞는 패스워드를 입력해줍니다. 초기 비밀번호인 ORACLE_PWD를 pass로 설정했었기 때문에, (ID : system  / pasword : pass)를 입력해줍니다.


## 실습테이블 구축

DBeaver에 무사히 연결을 하셨다면, 이제 테이블을 구축할 차례입니다. 

다행히도, Oracle 에서는 실습을 위한 테이블들을 다수 제공하고 있습니다. [Oracle 실습 테이블 구축 홈페이지](https://www.oracletutorial.com/getting-started/oracle-sample-database/) 에서 해당 내용들을 확인할 수 있습니다.

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/11.png "테이블 스키마")

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/12.png "테이블 정보 2")


링크를 타고 들어간 이후, Sample Database 파일을 다운로드 받으면 총 4개의 파일을 확인할 수 있습니다.

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/13.png "테이블 정보 3")

이후, 도커로 sqlplus에 접속하고
~~~bash
docker exec -it oracle19 sqlplus
~~~

먼저 ot_schema.sql 을 복사 붙여넣기 하여 실행해주면 Table Schema 를 일괄적으로 생성할 수 있습니다.

이후 ot_data.sql을 열어서

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/14.png "테이블 정보 4")

OT. 라는 string을 일괄적으로 빈 스페이스로 모두 바꿔주신 후 sqlplus에서 실행하면, 데이터를 모두 제대로 넣을 수 있습니다. (현재 로컬 DB에는 OT라는 테이블 스페이스가 없기 때문에 바꿔줘야 합니다!)

![800x400](../../../assets/img/post_img/mac에서%20Oracle%20실습환경%20구축하기%20(feat.%20Docker%20Desktop)/15.png "테이블 정보 15")

축하합니다! DB 구축과 테이블 설정 및 데이터 삽입까지 모두 완료했습니다!

## 참고한 글과 문서
* [https://shanepark.tistory.com/400](https://shanepark.tistory.com/400)
* [https://velog.io/@profile_exe/DB-oracle-database-19c-with-docker-arm-%EA%B8%B0%EB%B0%98-mac-%EC%84%A4%EC%B9%98-%EB%B0%8F-%EC%8B%A4%ED%96%89#2-vscode-docker-desktop-extension-%EC%84%A4%EC%B9%98](https://velog.io/@profile_exe/DB-oracle-database-19c-with-docker-arm-%EA%B8%B0%EB%B0%98-mac-%EC%84%A4%EC%B9%98-%EB%B0%8F-%EC%8B%A4%ED%96%89#2-vscode-docker-desktop-extension-%EC%84%A4%EC%B9%98)