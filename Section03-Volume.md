# 도커 볼륨 

- [도커 볼륨](#도커-볼륨)
  - [도커에서 사용되는 데이터](#도커에서-사용되는-데이터)
    - [Application Data](#application-data)
    - [Temporary Data](#temporary-data)
    - [Permanent Data](#permanent-data)
  - [Volumes](#volumes)
    - [Anonymous Volumes](#anonymous-volumes)
    - [Named Volumes](#named-volumes)
    - [Bind Mounts](#bind-mounts)
    - [예제 설명](#예제-설명)
    - [Volumes 요약](#volumes-요약)
  - [Read-only Volumes](#read-only-volumes)
  - [도커 볼륨 관리](#도커-볼륨-관리)
  - [COPY vs Bind Mount](#copy-vs-bind-mount)
  - [.dockerignore](#dockerignore)
  - [ARG](#arg)
  - [ENV](#env)

## 도커에서 사용되는 데이터

### Application Data 

애플리케이션 데이터는 애플리케이션에서 사용하는 소스 코드와 구성 파일을 의미합니다.

소스 코드는 개발자가 작성한 코드이며, 구성 파일은 애플리케이션의 설정을 저장하는 Dockerfile 입니다.

애플리케이션 데이터는 이미지에 저장되며, 수정할 수 없습니다.

### Temporary Data

임시 데이터는 컨테이너가 실행되는 동안 생성되는 데이터를 의미합니다.

컨테이너가 종료되면 임시 데이터는 삭제됩니다.

임시 데이터는 컨테이너의 파일 시스템에 저장되며, 컨테이너가 종료되면 삭제됩니다.

### Permanent Data

영구 데이터는 컨테이너가 실행되는 동안 생성되고 가져오는 데이터이지만,
임시 데이터와는 다르게 컨테이너가 종료되어도 유지되는 데이터를 의미합니다.

영구 데이터는 볼륨을 사용하여 컨테이너의 파일 시스템과 호스트의 파일 시스템을 연결하여 사용합니다.
볼륨은 컨테이너가 종료되어도 데이터를 유지할 수 있도록 도와줍니다.

## Volumes

도커 볼륨은 컨테이너에 마운트된 호스트 머신의 디렉토리입니다.

즉 컨테이너는 호스트 머신의 디렉토리에 매핑된 디렉토리를 사용하여 데이터를 영구적으로 저장합니다.

볼륨을 사용하여 데이터를 저장하면 컨테이너가 종료되어도 데이터가 유지됩니다.

### Anonymous Volumes

도커 볼륨을 생성할 때 이름을 지정하지 않으면 익명 볼륨이 생성됩니다.

익명 볼륨은 도커에서 자동으로 생성되며, 암호화되어 있습니다. 

익명 볼륨은 컨테이너가 종료되어도 데이터를 유지되지만,

외부에서 해당 볼륨에 접근할 수 없고,
컨테이너를 새로 생성하면 이전 컨테이너의 볼륨은 삭제됩니다.

### Named Volumes

도커 볼륨을 생성할 때 이름을 지정하면 명명된 볼륨이 생성됩니다.

명명된 볼륨은 컨테이너가 삭제되어도 데이터를 유지하며,
외부에서 해당 볼륨에 접근할 수 있습니다.

명명된 볼륨을 사용하기 위해서는 다음과 같이 명령어를 사용합니다.

```bash
$ docker run -v [volume-name]:[container-path] [image-name]
```

### Bind Mounts

바인드 마운트는 호스트 머신의 디렉토리를 컨테이너에 마운트하는 방법입니다.

바인드 마운트를 사용하면 호스트 머신의 디렉토리를 컨테이너의 디렉터리에 연결하여 데이터를 저장할 수 있습니다.

명명된 볼륨는 영구적인 데이터를 저장하지만 편집을 불가능한 반면, 
바인드 마운트는 영구적이고 **편집 가능한** 데이터를 저장할 때 유용합니다. 

바인드 마운트를 사용하기 위해서는 다음과 같이 명령어를 사용합니다.

```bash
$ docker run -v [host-path]:[container-path] [image-name]
```

### 예제 설명

강의 예제에서는 익명 볼륨, 명명된 볼륨, 바인드 마운트의 각 특징을 모두 활용합니다.

다음은 Section03 예제를 실행하는 명령어입니다.

```bash
$ docker run -p 3000:80 --rm -v "$(pwd):/app" -v feedback:/app/feedback -v /app/node_modules feedback-app:volumes
```

> 볼륨이 마운트되는 순서는 명령의 순서가 아니라 경로의 구체적인 순서 따라 결정됩니다.  
> 따라서, 위 명령어에서 `/app/node_modules` 디렉토리는 `/app` 디렉토리보다 우선적으로 마운트됩니다.  
> 이때 `/app/node_modules` 디렉토리를 볼륨에 마운트 하지 않으면 `npm install` 명령어로 생성된 컨테이너의 `/app/node_modules`가 삭제 혹은 변경됩니다.

위 명령어에서 각 볼륨은 다음과 같이 사용됩니다.

* `-v "$(pwd):/app"`: 바인드 마운트
  * 현재 디렉토리를 컨테이너의 `/app` 디렉토리에 연결합니다.
  * 이는 소스 코드를 수정할 때마다 컨테이너에 반영되도록 합니다.
  * `nodemon`과 같은 도구를 사용하면 소스 코드 변경시 컨테이너를 재시작 하지 않아도 변경사항을 반영할 수 있습니다.
* `-v feedback:/app/feedback`: 명명된 볼륨
  * `feedback`이라는 명명된 볼륨을 생성하고, 컨테이너의 `/app/feedback` 디렉토리에 연결합니다.
  * 이는 컨테이너가 삭제되어도 데이터를 유지하도록 합니다.
  * 이전에 생성된 `feedback` 볼륨이 있다면 해당 볼륨을 사용합니다.
* `-v /app/node_modules`: 익명 볼륨
  * 컨테이너의 `/app/node_modules` 디렉토리에 익명 볼륨을 생성합니다.
  * 이는 컨테이너가 종료되어도 데이터를 유지하도록 합니다.
  * 이를 설정하지 않으면 바인드 마운트 이후에 `npm install` 명령어로 생성된 컨테이너의 `/app/node_modules`가 삭제 혹은 변경될 수 있습니다.

### Volumes 요약

* 익명 볼륨
  * 컨테이너가 종료되어도 데이터 유지
  * 컨테이너를 새로 생성하면 이전 컨테이너의 볼륨은 삭제
  * 외부에서 접근 불가
  * 컨테이너 간 데이터 공유에 불가
  * 컨테이너에 존재하는 특정 데이터를 잠그는데 유용 (예: `/app/node_modules`)
* 명명된 볼륨
  * 컨테이너가 삭제되어도 데이터 유지
  * 컨테이너를 새로 생성해도 이전 컨테이너의 볼륨은 유지
  * 외부에서 접근 가능
  * 컨테이너 간 데이터 공유에 유용
* 바인드 마운트
  * 호스트 머신의 디렉토리를 컨테이너에 연결
  * 컨테이너가 삭제되어도 데이터 유지
  * 편집 가능한 데이터를 저장할 때 유용
  * 컨테이너 간 데이터 공유에 유용

## Read-only Volumes

읽기 전용 볼륨을 사용하면 바인드 마운트를 사용할 때 컨테이너가 호스트의 디렉토리에 쓰기 권한이 없도록 설정할 수 있습니다.

읽기 전용 볼륨은 바인드 바운트를 사용할 때 `ro` 옵션을 사용하여 설정할 수 있습니다.

```bash
$ docker run -v [host-path]:[container-path]:ro [image-name]
```

세부 경로 설정을 통해 읽기 전용 볼륨의 특정 디렉토리에 대해 쓰기 권한을 설정할 수 있습니다.

위에서 node_modules를 익명 볼륨으로 설정하여 컨테이너가 종료되어도 데이터를 유지하도록 설정한 것과 같이

세부적인 경로 설정을 통해 특정 디렉토리에 대한 쓰기 권한을 추가합니다.

```bash
$ docker run -v [host-path]:[container-path]:ro -v [container-path] [image-name]
```

위 명령어에서 두 번째 볼륨은 첫 번째 볼륨의 설정을 오버라이드하여 쓰기 권한을 부여합니다.

## 도커 볼륨 관리

도커 볼륨 목록을 확인하기 위해서 다음 명령어를 사용합니다.

```bash
$ docker volume ls
```

> 바인드 마운트된 볼륨은 도커에서 관리하지 않기 때문에 `docker volume ls` 명령어로 확인할 수 없습니다.

도커 볼륨은 도커 실행시 자동으로 생성되지만, 다음 명령어를 사용하여 볼륨을 수동으로 생성하거나 삭제할 수 있습니다.

```bash
$ docker volume create [volume-name]
```

```bash
$ docker volume rm [volume-name]
```

`prune` 명령어를 사용하여 사용하지 않는 볼륨을 삭제할 수 있습니다.

```bash
$ docker volume prune
```

`inspect` 명령어를 사용하면 볼륨의 상세 정보를 확인할 수 있습니다.

```bash
$ docker volume inspect [volume-name]
```

```json
[
    {
        "CreatedAt": "2024-06-26T03:01:48Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/feedback-files/_data",
        "Name": "feedback-files",
        "Options": {},
        "Scope": "local"
    }
]
```

위 상세 정보에서 Mountpoint는 실제 볼륨이 저장된 경로를 의미합니다.

하지만 이 경로는 호스트 머신 파일 시스템에 있는 실제 경로가 아니라 도커 가상 파일 시스템에 있는 경로입니다.

만약 볼륨을 읽기 전용으로 생성한 경우 Options에 `ro` 옵션이 추가됩니다.

```json
[
    {
        "CreatedAt": "2024-06-26T03:01:48Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/feedback-files/_data",
        "Name": "feedback-files",
        "Options": {
            "ro": "true"
        },
        "Scope": "local"
    }
]
```

## COPY vs Bind Mount

바인드 마운트를 사용하는데도 여전히 COPY 명령어를 사용하는 이유는
프로덕션 환경에서 스냅샷을 유지하기 위해서 입니다.

프로덕션 환경에서 바인드 마운트를 사용하면 소스 코드를 수정할 때마다 컨테이너에 반영되기 때문에
컨테이너가 예상치 못한 방식으로 동작할 수 있습니다.

따라서, 프로덕션 환경에서는 소스 코드를 수정할 때마다 컨테이너에 반영되지 않도록 COPY 명령어를 사용하여
스냅샷을 유지하는 것이 좋습니다.

개발 환경에서는 COPY 명령어를 유지한 상태로
바인드 마운트를 사용하여 소스 코드를 수정할 때마다 컨테이너에 반영하게 할 수 있습니다.

## .dockerignore

`.dockerignore` 파일을 사용하여 COPY 명령어를 사용할 때 복사하지 않을 파일을 지정할 수 있습니다.

`.dockerignore` 파일은 `.gitignore` 파일과 유사하게 사용할 수 있습니다.

```bash
# .dockerignore
.git/
node_modules/
npm-debug.log
```

## ARG

`ARG` 명령어는 **빌드** 시간에 사용되는 변수를 정의할 때 사용합니다.
`${변수명}` 형식으로 변수를 사용할 수 있습니다.

```dockerfile
ARG NODE_VERSION=14.17.0
FROM node:${NODE_VERSION}
```

또한 `docker build` 명령어로 이미지를 빌드할 때 `--build-arg` 옵션을 사용하여 변수를 전달할 수 있습니다.

```dockerfile
FROM node:${NODE_VERSION}
```

```bash
$ docker build --build-arg NODE_VERSION=14.17.0 -t feedback-app:volumes .
```

## ENV

`ENV` 명령어는 **런타임**에 사용되는 환경 변수를 정의할 때 사용합니다.
`Dockerfile`에 `ENV` 명령어로 설정하거나 도커 실횅시 `-e` 옵션을 사용하여 런타임에 환경 변수를 전달할 수 있습니다.

```dockerfile
ENV NODE_ENV production
```

```bash
$ docker run -e NODE_ENV=development feedback-app:volumes
```

만약 도커파일에서 설정한 환경변수를 사용하고 싶으면 앞에 `$`를 붙여서 사용합니다.

```dockerfile
ENV PORT 3000
EXPOSE $PORT
```

`.env` 파일을 사용하여 환경 변수를 설정하고 싶으면 `--env-file` 옵션을 사용합니다.

```bash
PORT=3000
```

```bash
$ docker run --env-file .env feedback-app:volumes
```

> 보안사항  
> `credential` 혹은 `access key`와 같은 중요한 정보는 `Dockerfile`에 명시하지 않고,
> `docker run` 명령어를 실행할 때 옵션으로 추가하거나, `.env` 파일을 사용하여 환경 변수를 설정하는 것이 좋습니다.  
> `Dockerfile`에 환경 변수를 명시하면 이미지에 환경 변수가 포함되기 때문에
> `docker history` 명령어로 이미지의 환경 변수를 확인할 수 있기 때문입니다.





