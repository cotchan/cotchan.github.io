---
layout: post
title: docker-compose를 사용한 spring-boot 무중단 배포 과정
subtitle: with git-action & S3 & CodeDeploy & docker-compose
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### INTRO

- 프로젝트를 진행하며 멀티 모듈 구조인 서로 다른 서버 3개를 한 번에 띄웠었습니다.
- 그래서 도커 컴포즈를 이용했었는데, 도커 컴포즈도 이용하면서 무중단 배포를 적용해야겠다 싶어 공부한 내용을 정리합니다.
- 여기서 적용한 무중단 배포는 `1대의 서버`내에서 `nginx reload`를 사용하여 서버가 서비스를 제공하는 인스턴스를 변경하는 것을 의미합니다!
- +) `Blue/Green` 배포 방식을 적용했습니다.

### 2개의 docker-compose

- 우선, `blue & green` 배포를 위해 docker-compose.yml을 `2개 정의`합니다.
  - `docker-compose.blue.yml`
  - `docker-compose.green.yml`
- 두 파일은 `외부에 개방하는 포트정보만 다를 뿐` 나머지 `컨테이너가 만들어지는데 주입받는 정보는 동일`합니다.

#### blue

```yml
# docker-compose.blue.yml
version: "3"

services:
  api_server:
    container_name: api-blue
    build:
      context: .
      dockerfile: ./Dockerfile
    ports:
      - "8081:8080"
```

#### green

```yml
# docker-compose.green.yml
version: "3"

services:
  api_server:
    container_name: api-green
    build:
      context: .
      dockerfile: ./Dockerfile
    ports:
      - "8082:8080"
```

### nginx 설정

- 무중단 배포를 위해 nginx 설정을 하는 방법은 다양합니다.
- 다만 제가 적용해본 방식은 프로젝트 내에 별도로 blue, green URL PATH를 주고, 배포하는 서버의 nginx proxy 설정에 추가로 반영하는 방식으로 reload가 적용되게 하였습니다.
- 즉, `service-url-blue.inc`, `service-url-green.inc`을 각각 정의하고 이를 `/etc/nginx/conf.d/` 경로에 복사함으로써 nginx가 설정을 읽을 수 있도록 했습니다.
  - 정확한 적용방식은 아래 배포 스크립트 부분에서 나옵니다.

```conf
  include /etc/nginx/conf.d/*.conf;
  include /etc/nginx/sites-enabled/*;

  server {
      include /etc/nginx/conf.d/*.inc;
      listen 80;
      server_name SERVER_NAME;

      location / {
          proxy_pass $service_url;
      }
  }
```

#### service-url-blue.inc

- `blue`가 서비스 인스턴스로 떴을 때 매핑시킬 포트 정보

```
set $service_url http://127.0.0.1:8081;
```

#### service-url-green.inc

- `green`이 서비스 인스턴스로 떴을 때 매핑시킬 포트 정보

```
set $service_url http://127.0.0.1:8082;
```

### 배포 스크립트

- 이제 실제로 서버에서 실행되어 `blue/green 배포`를 실행해주는 배포스크립트 `deploy.sh` 입니다.
- 우선 전체 소스코드와 함께 참고했던 내용에서 수정한 부분들을 살펴보겠습니다.
- 전체적인 흐름은 아래와 같습니다.
  - `Blue`를 기준으로 현재 떠 있는 컨테이너를 확인합니다.
    - `Blue`가 떠있을 시 => `Green`을 배포합니다.
    - `Green`가 떠있을 시 => `Blue`를 배포합니다.
  - 새로 배포한 컨테이너가 `Up` 상태로 제대로 배포가 되었다면, 기존에 떠있던 컨테이너를 `docker-compose down` 으로 삭제합니다.

```bash
#!/bin/bash
# deploy.sh

ABSPATH=$(readlink -f $0) # 현재 파일 위치 절대 경로로 받기
ABSDIR=$(dirname $ABSPATH) # 현재 디렉토리 위치 /spring-boot/scripts
SOURCE_REPOSITORY=$(dirname $ABSDIR) # 부모 디렉토리 위치 /spring-boot

DOCKER_APP_NAME=spring-boot # DOCKER_APP_NAME에는 프로젝트명을 적는다.

# Blue 를 기준으로 현재 떠있는 컨테이너를 한다.
EXIST_BLUE=$(docker-compose -p ${DOCKER_APP_NAME}-blue -f ${SOURCE_REPOSITORY}/docker-compose.blue.yml ps | grep Up)

# 컨테이너 스위칭
if [ -z "$EXIST_BLUE" ]; then
    echo "blue up"
    docker-compose -p ${DOCKER_APP_NAME}-blue -f ${SOURCE_REPOSITORY}/docker-compose.blue.yml up --build -d
    BEFORE_COMPOSE_COLOR="green"
    AFTER_COMPOSE_COLOR="blue"
else
    echo "green up"
    docker-compose -p ${DOCKER_APP_NAME}-green -f ${SOURCE_REPOSITORY}/docker-compose.green.yml up --build -d
    BEFORE_COMPOSE_COLOR="blue"
    AFTER_COMPOSE_COLOR="green"
fi

sleep 10

# 새로운 컨테이너가 제대로 떴는지 확인
EXIST_AFTER=$(docker-compose -p ${DOCKER_APP_NAME}-${AFTER_COMPOSE_COLOR} -f ${SOURCE_REPOSITORY}/docker-compose.${AFTER_COMPOSE_COLOR}.yml ps | grep Up)
if [ -n "$EXIST_AFTER" ]; then
    echo "nginx reload.."
    # nginx.config를 컨테이너에 맞게 변경해주고 reload 한다
    sudo cp ${SOURCE_REPOSITORY}/nginx/service-url-${AFTER_COMPOSE_COLOR}.inc /etc/nginx/conf.d/backend-url.inc
    sudo systemctl reload nginx

    # 이전 컨테이너 종료
    echo "$BEFORE_COMPOSE_COLOR down"
    docker-compose -p ${DOCKER_APP_NAME}-${BEFORE_COMPOSE_COLOR} -f ${SOURCE_REPOSITORY}/docker-compose.${BEFORE_COMPOSE_COLOR}.yml down
else
    echo "> The new container did not run properly."
    exit 1
fi
```

#### DOCKER_APP_NAME

- 도커 컴포즈로 한 번에 배포하는 컨테이너들을 하나의 이름을 가진 프로젝트 묶어 식별하기 위해 정의합니다.

#### SOURCE_REPOSITORY

- 소스 코드가 위치하는 루트 경로입니다.
- 저 같은 경우는 `deploy.sh`는 `/script/deploy.sh`에 두고 `docker-compose.XXX.yml` 파일은 루트경로에 뒀습니다.
- 상대 경로로 조회하면 찾지를 못하다보니 절대 경로를 정의해주었습니다.

#### --build 옵션

- `docker-compose up --build`에서 `--build` 옵션은 선택사항입니다.
- 해당 옵션을 적용하면 이미지가 있든 없든 이미지를 빌드하고 컨테이너를 시작합니다.
- 이 옵션을 적용한 이유는 컨테이너를 한 번 실행한 후 소스 코드를 수정하고 다시 도커 컴포즈를 이용해 컨테이너를 시작할 때는 변경사항이 적용되어야 하므로 이 명령어로 컨테이너를 실행해야 합니다.
- 소스 코드는 프리징된 상태라면 `--build` 옵션은 필요없습니다.

#### nginx reload 부분

- 아까 정의했던 `service-url-blue.inc`, `service-url-green.inc`을 현재 뜨는 `blue/green`에 맞게 엔진엑스에 넣어주는 부분입니다.
- 해당 설정은 `backend-url.inc`라는 이름으로 들어가게 됩니다.

```bash
sudo cp ${SOURCE_REPOSITORY}/nginx/service-url-${AFTER_COMPOSE_COLOR}.inc /etc/nginx/conf.d/backend-url.inc
sudo systemctl reload nginx
```

- 위에서 추가해주는 `backend-url.inc` 파일 정보를 읽을 수 있도록 아까 위에서 `/etc/nginx/nginx.conf` 설정을 해줬습니다.

```
include /etc/nginx/conf.d/*.inc;
```

- 그리고 바꾼 설정이 적용되도록 nginx를 `reload`하면 됩니다.

```bash
sudo systemctl reload nginx
```

### appspec.yml

- 마무리 단계입니다.
- 저는 `AWS CodeDeploy`를 사용했기 때문에 빌드된 소스 코드에 대해 `deploy.sh`를 CodeDeploy Agent가 실행합니다.
- 그러면 위에서 써놓은 과정들이 실행되며 `blue/green` 방식의 무중단 배포가 진행됩니다.

```yml
version: 0.0
os: linux
files:
  - source:
    destination:
    overwrite:

permissions:
  - object: /
    pattern: "**"
    owner: ubuntu
    group: ubuntu

hooks: # CodeDeploy의 배포에는 각 단계 별 수명 주기가 있는데, 수명 주기에 따라 원하는 스크립트를 수행가능
  ApplicationStart:
    - location: scripts/deploy.sh
      timeout: 180
      runas: ubuntu
```

#### 추신

- 무중단 배포의 개념을 이해하고 적용하는 과정에서 아래 출처들이 많은 도움이 많이 되었습니다.
- 더 자세한 내용이 잘 나와있으니 꼭 보시는 걸 추천드립니다 :)

---

- 참고
  - [Github Actions + CodeDeploy + Nginx 로 무중단 배포하기 (1)](https://wbluke.tistory.com/39)
  - [Github Actions + CodeDeploy + Nginx 로 무중단 배포하기 (2)](https://wbluke.tistory.com/40)
  - [Github Actions + CodeDeploy + Nginx 로 무중단 배포하기 (3)](https://wbluke.tistory.com/41)
  - [[AWS] Spring Boot,Travis CI, CodeDeploy, Nginx로 무중단 배포하기](https://devlog-wjdrbs96.tistory.com/309)
  - [[AWS] Spring Boot, Nginx, Docker 로 무중단 배포하기 - 1탄](https://devlog-wjdrbs96.tistory.com/316)
  - [[AWS] Spring, Nginx, Docker로 무중단 배포하기 - 2탄](https://devlog-wjdrbs96.tistory.com/317)
  - [docker-compose 무중단 배포 1편 (blue, green)](https://jay-ji.tistory.com/m/99)
  - [spring boot, docker, docker-copmpose, nginx 이용해서 무중단 배포하기](https://kukekyakya.tistory.com/entry/spring-boot-docker-docker-compose-nginx-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C-%EB%AC%B4%EC%A4%91%EB%8B%A8-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0)
  - [[Docker] Docker Compose 프로젝트 무중단 배포](https://ozofweird.tistory.com/entry/Docker-Docker-Compose-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EB%AC%B4%EC%A4%91%EB%8B%A8-%EB%B0%B0%ED%8F%AC)
