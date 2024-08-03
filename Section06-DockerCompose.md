# Docker Compose

- [Docker Compose](#docker-compose)
  - [예제 - mongodb](#예제---mongodb)
  - [docker compose up \& down](#docker-compose-up--down)
  - [예제 - backend](#예제---backend)
  - [예제 - frontend](#예제---frontend)

지난 다중 컨테이너 환경을 구성하는 과정에서 수많은 docker build, docker run 명령어와 수십가지의 옵션을 사용했습니다.

docker compose는 이러한 명령어를 하나의 설정 파일로 관리할 수 있게 해줍니다.

docker compose는 Dockerfile을 대체하거나 Image, Container를 대체하는 것이 아닙니다. docker compose는 설정 파일을 통해 여러 컨테이너를 실행하고 관리하는 도구입니다.

> docker compose는 하나의 호스트에서 여러 컨테이너를 실행할 때 적합합니다. 다수의 호스트에서 컨테이너를 실행할 때는 주로 Kubernetes를 사용합니다.

## 예제 - mongodb

```yaml
version: "3.8"

services:
  mongodb:
    image: "mongo"        # Official MongoDB image in DockerHub
    volumes:
      - data:/data/db     # Named Volume
    env_file:
      - ./env/mongo.env

volumes:
  data:
```

docker compose에서는 --rm -d 옵션을 기본값으로 사용하기 때문에 명시하지 않아도 됩니다.

네트워크 또한 새로운 네트워크를 자동으로 생성하고 모든 서비스를 그 네트워크에 추가합니다. 즉, 동일한 docker compose 설정 파일로 정의된 서비스는 동일한 도커 네트워크에 속하는 것입니다.

위 예제에서 mongodb 서비스는 data라는 Named Volume을 사용합니다. docker compose에서 Named Volume은 volumes 섹션에 정의합니다.

## docker compose up & down

작성한 docker compose 설정 파일을 실행하려면 다음 명령어를 사용합니다.

```bash
$ docker-compose up
```

docker compose는 docker-compose.yml 파일을 찾아서 실행합니다. 파일 이름이 다르다면 -f 옵션을 사용하여 파일을 지정할 수 있습니다.

```bash
$ docker-compose -f docker-compose.yml up
```

detach 모드로 실행하려면 -d 옵션을 사용합니다.

```bash
$ docker-compose up -d
```

docker compose로 실행한 컨테이너를 중지하려면 다음 명령어를 사용합니다.

```bash
$ docker-compose down
```

docker compose로 생성한 볼륨을 삭제하려면 -v 옵션을 사용합니다.

```bash
$ docker-compose down -v
```

## 예제 - backend

```yaml
version: "3.8"

services:
  # ...

  backend:
    build: ./backend      # Directory containing Dockerfile
    ports:
      - "80:80"
    volumes:
      - logs:/app/logs    # Named Volume
      - ./backend:/app    # Bind Mount  
      - /app/node_modules # Anonymous Volume
    env_file:
      - ./env/backend.env
    depends_on:
      - mongodb

volumes:
  logs:
```

위 예제에서 backend 서비스는 logs라는 Named Volume과 ./backend 디렉토리를 /app 디렉토리에 Bind Mount하고 /app/node_modules를 Anonymous Volume로 사용합니다.

docker compose에서는 depends_on 옵션을 사용하여 다른 서비스가 실행되기 전에 해당 서비스를 실행할 수 있습니다.

docker compose up 명령어를 사용해서 컨테이너를 실행하면 위에 정의한 서비스 이름과 `다른 이름으로 컨테이너 생성`된 것을 볼 수 있습니다.

docker compose에서 `서비스의 이름으로 mongodb로 설정했기 때문에` 여전히 mongodb로 접근할 수 있습니다. 즉, 도커 네트워크에서 서비스 이름으로 접근할 수 있습니다.

## 예제 - frontend


```yml
version: "3.8"

services:
  # ...

  frontend:
    build: ./frontend     # Directory containing Dockerfile
    ports:
      - "3000:3000"
    volumes:
      - ./frontend/src:/app/src   # Bind Mount
    stdin_open: true      # Keep STDIN open even if not attached
    tty: true             # Allocate a pseudo-TTY for interactive sessions
    depends_on:
      - backend
```

이제 docker compose up 명령어를 사용하면 mongodb, backend, frontend 서비스가 동시에 실행됩니다.