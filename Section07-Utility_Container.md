# Utility Contiainer

- [Utility Contiainer](#utility-contiainer)
  - [Utility Contiainer를 사용하는 이유?](#utility-contiainer를-사용하는-이유)
  - [Node.js 유틸리티 컨테이너 만들기](#nodejs-유틸리티-컨테이너-만들기)
    - [exec 명령어 사용](#exec-명령어-사용)
    - [default command 사용](#default-command-사용)
    - [ENTRYPOINT 사용](#entrypoint-사용)
    - [docker-compose 사용](#docker-compose-사용)

지금까지 도커를 머플리케이션 컨테이너 활용해왔습니다.

이번에는 유틸리티 컨테이너로서 활용하는 방법을 소개합니다.

유틸리티 컨테이너랑 어플리케이션 코드 없이 Node.js, php와 같은 특정 환경만을 제공하는 컨테이너입니다.

## Utility Contiainer를 사용하는 이유?

예를 들어, Node.js를 사용하는 경우 처음 프로젝트를 생성하려면 npm init 명령어를 사용합니다.

하지만 npm init 명령어를 사용하려면 Node.js가 설치되어 있어야 합니다.

도커의 장점은 Node.js를 설치하지 않아도 컨테이너 환경에서 Node.js를 사용할 수 있다는 것입니다.

하지만 프로젝트를 생성하기 위해 Node.js를 설치하는 것은 비효율적입니다.

유틸리티 컨테이너는 호스트 머신에 부가 도구를 설치하지 않고 특정 환경을 사용할 수 있다는 장점이 있습니다.

이런 경우 유틸리티 컨테이너를 사용하면 편리합니다.

## Node.js 유틸리티 컨테이너 만들기

### exec 명령어 사용

docker exec 명령어를 사용하면 컨테이너 밖에서 컨테이너 내부 명령어를 실행할 수 있습니다.

먼저 node 컨테이너 -it -d 옵션을 사용하여 실행합니다.

```bash
$ docker run -it --d --name node node:21
```

다음 명령어를 사용하여 컨테이너 내부에서 명령어를 실행합니다.

```bash
$ docker exec -it node npm init
```

### default command 사용

exec 명령어 대신 run 명령어를 사용할 때 default command를 사용할 수 있습니다.

```bash
$ docker run -it --name node node:21 npm init
```

추가로 -v 옵션을 사용하여 호스트 머신의 디렉토리를 컨테이너에 마운트할 수 있습니다.

```bash
docker run -it -v $PWD:/app node-util npm init
```

### ENTRYPOINT 사용

또한 Dockerfile의 ENTRYPOINT를 사용하면 컨테이너를 실행할 때 작성한 default command가 ENTRYPOINT에 설정된 명령어 뒤에 추가됩니다.

```Dockerfile
ENTRYPOINT ["npm"]
```

```bash
docker run -it -v $PWD:/app node-util install express --save
```

### docker-compose 사용

docker-compose를 사용하면 추가적인 옵션을 사용하지 않고도 컨테이너를 실행할 수 있습니다.

```yml
version: "3.8"

services:
  npm:
    build: ./
    stdin_open: true
    tty: true
    volumes:
      - ./:/app
```

docker-compose run 명령어는 docker-compose.yml에 정의된 서비스를 실행합니다.
이 떄 이름을 지정하면 해당 서비스만 실행됩니다.

```bash
$ docker-compose run npm init
```

--rm 옵션을 사용하면 컨테이너 종료시 컨테이너가 삭제됩니다.

```bash
$ docker-compose run --rm npm init
```
