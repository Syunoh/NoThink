## 도커 컨테이너 

  ### 도커 컨테이너 생성 
  
    $docker create [image]

  - docker create 명령어를 입력하면, 도커엔진이 로컬 호스트에서 이미지 정보를 찾아서 컨테이너 생성
  - 만약, 로컬 호스트에 이미지가 없을 경우에는 도커엔진이 자동으로 docker pull을 실행하여 Docker Repoistory에서
    이미지를 가져오는 것을 시도

  ### 도커 컨테이너 시작
  
  **docker start - 도커 컨테이너가 생성되어있다면 시작**
     
    $ docker start [container]

  **docker run - 사실상 docker create 와 start를 합친 명령어**

    $ docker run [option] [image] [command] [args]

  **run [option]**

    1. -i 
      컨테이너와 상호 입출력을 가능하게 합니다.
    2. -t
      터미널 명령을 정상적으로 실행할 수 있도록 설정

    (보통 -it 으로 실행)

    3. --rm 
      컨테이너가 실행 종류될 경우 이미지 자동으로 삭제
    4. -d
      detached모드로 실행(기본적으로 Foreground에서 동작) -it가 컨테이너 내부에 attached 모드로 진입했던 것과 반대
    5. -name 
      컨테이너 이름 설정
    6. -v
      호스트 운영체제와 컨테이너 사이의 파일시스템 디렉토리를 마운트하기 위한 옵션(볼륨에서 다룰 예정)

  ### 도커 컨테이너 종료

  **docker stop - 컨테이너를 안전하게 종료**

    $ docker stop [container]

  **모든 컨테이너 종료**

    $ docker stop ${docker ps -a -q)

  ### 도커 컨테이너 목록 확인

  **실행중인 컨테이너 상태 확인**

    $ docker ps

  **전체 확인 (stop 되어있는 컨테이너 포함)**

    $ docker ps -a

  **컨테이너 상세 정보 확인**

    $ docker inspect [container]

  ### 도커 컨테이너 삭제

  **컨테이너 삭제**

    $ docker rm [container]

  **컨테이너 실행 종료 후 자동 삭제**

    $ docker run --rm [container]

  **중지된 모든 컨테이너 삭제**

    $ docker container prume

   <br>
   <br>
   <br>
   
## 도커 볼룸

  
![image](https://github.com/Syunoh/StoneBox/assets/100738448/0b1f703e-a8a6-40f0-8028-5b180a226f98)


  - 위 이미지는 레이어 구조로 되어있는데, Dokcerfile 내에 작성되어있는 여러 명령어들이 순차적으로 레이어가 쌓이듯이 저장된다고 보면 됨

    Layer 1: Base 우분투L레이어 설치
    Layer 2: 우분투 운영체제에서 필요로 하는 패키지들을 설치 (apt-get 패키지 설치하는 것들)
    Layer 3: 패키지 설치 (파이썬 패키지 설치하는 예시)
    Layer 4: 소스코드를 복사를 한다면 복사할 내용이 Layer4에 저장 (소스코드 수정)
    Layer 5: Entrypoint 업데이트

  - 도커 컨테이너로 docker run 명령어로 이미지를 실행하면 컨테이너가 되는데, 여기서 두 가지 레이어를 가지게 됨
    첫번째는 "이미지 계층", 두번째는 "컨테이너 계층"
    이미지 계층에서 이미지는 항상 동일하고 Read Only로 사용하게 됨. 변경사항을 적용할 수 없음.
    컨테이너 계층은 컨테이너 상에서 새로운 파일을 쓴다거나 할 때는 기본적으로 Container Layer에 파일을 쓰게 되는데, Read Write권한      이 있음.

    ![image](https://github.com/Syunoh/StoneBox/assets/100738448/5cbbac15-8977-4dbe-a746-9953559d226f)


  ### 호스트 볼륨 공유하기

  **호스트 운영체제의 디렉토리를 컨테이너 내에 마운트 시키는 작업**

  아래 명령어를 입력해서 mysql 데이터베이스 컨테이너를 실행
  
    $ docker run -d --name wordpressdb_hostvolume \
    -e MYSQL_ROOT_PASSWORD=password \
    -e MYSQL_DATABASE=wordress \
    -v /home/wordpress_db:/var/lib/mysql \
    mysql:5.7

  아래 명령어를 입력해서 워드프레스 웹 서버 컨테이너를 생성

    $ docker run -d \
    -e WORDPRESS_DB_PASSWORD=password \
    --name wordpress_hostvolume \
    --link wordpressdb_hostvolume:mysql \
    -p 80 \
    wordpress

  워드프레스 컨테이너에 -p 옵션으로 컨테이너의 80포트를 외부에 노출했으므로 docker ps 명령어에서 확인한 wordpress_hostvolume컨테이 
  너의 호스트 포트로 워드프레스 컨테이너에 접속

  ### 볼륨 컨테이너

  ![image](https://github.com/Syunoh/StoneBox/assets/100738448/3aae7cf4-2b6a-4ffd-b22c-85ad03cbefe4)

  - 볼륨 컨테이너를 이용하는 방법은 볼륨을 특정 어플리케이션 컨테이너에서 마운트 시키는 것이 아니라, **볼륨 컨테이너에서 볼륨 마운트만 진행을 하고 아무것도 하지 않는 컨테이너를 만들어서 어플리케이션 컨테이너가 볼륨 컨테이너를 참조하게 해서 마치 볼륨 컨테이너같이 사용**할 수 있음. 그러면 Data-only Container의 마운트 목록을 어플리케이션 컨테이너가 공유받을 수 있음.

  먼저 호스트 볼륨을 공유하는 volume_override 라는 이름의 컨테이너를 생성
  
    $ docker run -d \
    -it \
    --name volume_override \
    -v $(pwd)/html:/usr/share/nginx/html \
    ubuntu:focal

  아래 명령어를 입력하여 --volumes-from 옵션을 사용하여 volume_override 컨테이너에서 볼륨을 공유받는 컨테이너를 생성

    $ docker run -d \
    --name user_nginx1 \
    --volumes-from volume_override \
    -p 80:80 \
    nginx

    $ docker run -d \
    --name user_nginx2 \
    --volumes-from volume_overrdie \
    -p 8080:80 \
    ngnix

  ![image](https://github.com/Syunoh/StoneBox/assets/100738448/54fa06f2-1cef-44a4-ac5a-d35a8bce5bb4)

  **--volumes-from** 옵션을 적용한 컨테이너와 볼륨 컨테이너 사이의 관계
  이러한 구조를 활용하여 호스트에서 볼륨만 공유하고 별도의 역할을 하지 않는 '볼륨 컨테이너'로 활용해서 컨테이너 활용 가능

  docker exec 명령어로 user_nginx1 컨테이너에 접속해서 /usr/share/nginx/html에 마운트가 잘되어있는지 확인 가능

    $ docker exec -it bfaf bash


  ### 도커 볼륨

  - **도커가 제공하는 볼륨관리 기능을 통해서 볼륨을 생성하고 삭제하고 관리**를 할 수 있음.

  - 이렇게 생성된 볼륨은 도커가 특정 호스트 경로에 데이터가 저장됨.

  #### 도커 볼륨 구조

  ![image](https://github.com/Syunoh/StoneBox/assets/100738448/da43d19b-9c0e-4a2d-a65d-014fdd844408)

  **도커 볼륨 생성**

     $ docker volume create --name [도커 볼륨 이름]

   **도커 볼륨 목록 확인**
    
    $ docker volume ls

  **도커 볼륨 정보 확인 - 마운트 경로등의 볼륨 정보를 확인 가능**

    $ docker volume inspect [도커 볼륨 이름]

<br>   
<br>
<br>
         
## 도커 이미지

  ![image](https://github.com/Syunoh/StoneBox/assets/100738448/52841631-be5a-4706-9d45-9421d1c2e1a2)

  - 도커 이미지는 Layer 구조로 이루어져 있음. Layer 구조를 사용하므로 기존 base layer는 변경할 필요없이 새로운 Layer를 추가만 하면 됨.

  - ubuntu 이미지를 베이스로 nginx 이미지를 만들고 web app 이미지를 nginx 이미지를 베이스로 만들어 레이어를 구성
    만약, 나중에 web app 이미지의 소스를 수정한다고 하면 web app source 레이어만 다운받으면 되기 때문에 효울적

  ### 도커 이미지 생성

      $ docker search [image name]

  - 하지만, 컨테이너에 특정 개발 환경을 직접 구축한 뒤 사용자만의 이미지를 직접 생성하야 하므로 컨테이너에서 이미지를 만드는 법

  - 먼저 우분투 focal 이라는 컨테이너를 생성
    
        $ docker run -it --name user_ubuntu ubuntu:focal

  - 컨테이너 내부에 my_name이라는 파일 생성하고 저장해서 기존의 이미지에서 변경사항을 만듦.

        $ echo user >> my_name

    ![image](https://github.com/Syunoh/StoneBox/assets/100738448/b0844058-2612-4333-8ed4-d03165fcf5c1)

  - 그런 다음 컨테이너에서 도커 호스트로 나와서 docker commit으로 컨테이너 이미지 생성

        $ docker commit [options] [container] [repository:tag]

  - 아래의 명령어는 user_ubuntu 컨테이너를 user_ubuntu:first라는 이름의 이미지로 생성

        $ docker commit \
        -a "username" -m "my first commit" \
        user_ubuntu \
        user_ubuntu:first

  - **-a 옵션**: 이미지의 작성자를 나타냄
  - **-m 옵션**: 이미지 커밋에 포함될 커밋 메세지

  - 저장소 이름을 입력하지 않아도 상관없지만 입력하지 않으면 자동으로 latest로 설정됨.

  - docker images로 컨테이너에 이미지가 생성되었는지 확인

  ![image](https://github.com/Syunoh/StoneBox/assets/100738448/2fadaee8-d755-4e85-a13e-8c046d644dd7)

  - docker images insepct 명령어로 두 개의 이미지의 레이어들을 비교

  ![image](https://github.com/Syunoh/StoneBox/assets/100738448/efcad6c9-ee08-4d16-a285-efc31bd37381)

  - ubuntu:focal 이미지를 베이스로 만든 ubuntu:first 이미지의 레이어는 기존에 존재하던 focal 에서 해시값을 가진 이미지 레이어에 추가해서 가진 것을 확인 가능

  ### 도커 이미지 삭제

    $ docker rmi [repository name:tag]

  - 만약, 이미지를 사용중인 컨테이너가 존재한다면 이미지를 삭제할 수 없음. (컨테이너 docker stop 명령어를 사용한 후 이미지 삭제)

    ![image](https://github.com/Syunoh/StoneBox/assets/100738448/4452a684-7d0d-45a1-abdb-4a8718e89d5e)

  - first 이미지를 삭제하는 명령어

    ![image](https://github.com/Syunoh/StoneBox/assets/100738448/170a4d84-bb8f-40e9-a64f-3cb14da30c7b)

  - docker images를 통해 컨테이너에서 삭제된 것을 확인 가능

<br>
<br>
<br>
<br>

## Dokerfile 

  ### 도커파일로 이미지 생성

  ![image](https://github.com/Syunoh/StoneBox/assets/100738448/e2de437c-9775-4972-ae25-40237b1e2dcd)

  - 어플리케이션을 컨테이너화하기 위한 장기적인 시점에서 보았을 때, 도커 파일을 작성하는 것은 이미지를 생성하는 방법을 기록하는 것 뿐만 아니라 이미지의 빌드, 베포 측면에서도 매우 유리 (이미지 생성을 자동화할 수 있고 쉽게 베포가 가능)

  ### Dokerfile 명령어

  1. FROM - 생성할 이미지의 베이스가 될 이미지를  결정하는 명령어. 반드시 한 번 이상 입력해야 하며 FROM 지시어를 시작으로 작성
  
  2. LABEL - 이미지의 메타데이터를 설정하는 명령어. (키값)

  3. WORKDIR - 명령어를 실행할 디렉토리를 지정하는 명령어.  (cd 명령어와 동일)

  4. EXPOSE - Dokerfile의 빌드로 생성된 이미지에서 노출할 포트를 지정하는 명령어. 해당 이미지를 사용하는 사용자에게 해당 포트를 사용한다고 문서화하는 목적

  5. COPY - 현재 디렉토리 경로의 모든 파일과 디렉토리를 해당 디렉토리에 모두 복사하는 명령어

  6. RUN - 이미지를 만들기 위해 컨테이너 내부에서 명령어를 실행 (apt update, apt-get install apache2 명령어를 지정해주면 아파치 웹 서버가 설치된 이미지가 생성)

  7. ADD - 파일을 이미지에 추가

  8. CMD - 해당 이미지로 컨테이너를 실행할 때 어떤 명령어를 수행할 것인지를 결정하는 명령어. Dockerfile에서 한 번만 사용 가능

  9. ENTRYPOINT - 도커 컨테이너가 실행할 때 고정적으로 실행되는 스크립트 혹은 명령어

  ### Dockerfile로 이미지 빌드
  
  ![화면 캡처 2024-04-07 191902](https://github.com/Syunoh/StoneBox/assets/100738448/9cfa14f6-3826-4330-886a-574b69bbb9bb)

  - From - 노드 JS 16버전의 이미지를 가져옴
  
  - WORKDIR - 명령어를 실행할 위치를 /app/ 디렉토리로 이동
 
  - COPY - 호스트 운영체제의 현재 디렉토리 상에 있는 package로 시작해서 .json으로 끝나는 모든 파일을 복사해서 이미지상의 경로로 현재 디렉토리에다가 전부 복사
 
  - RUN - npm install을 지정해 주었으므로 이미지 상에서 WORKDIR 명령어로 지정해준 현재위치인 /app/ 디렉토리에서 복사하고 패키지 설치

  - EXPOSE - 8080 포트를 사용하겠다고 선언

  - CMD - 쉘에서 'node server.js'명령어를 실행

    <br>

  - 아래의 명령어로 이미지 생성

        $ docker build --force-rm -t nodejs-server:1.0.0 .

  - 다음으로 아래의 명령어를 입력해서 생성된 이미지로 컨테이너 실행

        $ docker run -d -P --name my_nodejs_server nodejs-server:1.0.0

    

  
