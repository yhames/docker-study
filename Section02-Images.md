# 도커 이미지와 컨테이너

- [도커 이미지와 컨테이너](#도커-이미지와-컨테이너)
  - [도커 이미지](#도커-이미지)
    - [도커 이미지란?](#도커-이미지란)
    - [레이어 기반 아키텍처](#레이어-기반-아키텍처)
    - [요약 정리](#요약-정리)
  - [도커 이미지와 컨테이너 관리](#도커-이미지와-컨테이너-관리)
    - [컨테이너 실행 및 중지](#컨테이너-실행-및-중지)
    - [컨테이너 Attach 및 Detach](#컨테이너-attach-및-detach)
    - [컨테이너 인터렉티브 모드](#컨테이너-인터렉티브-모드)
    - [컨테이너 삭제](#컨테이너-삭제)
    - [중지된 컨테이너 자동 삭제](#중지된-컨테이너-자동-삭제)
    - [이미지 Inspect](#이미지-inspect)
    - [컨테이너 파일 복사](#컨테이너-파일-복사)
    - [이미지와 컨테이너 이름 및 태그 지정](#이미지와-컨테이너-이름-및-태그-지정)

## 도커 이미지

### 도커 이미지란?

도커에서 컨테이너는 소프트웨어의 실행 단위입니다.

도커 이미지는 컨테이너를 생성하기 위한 템플릿 혹은 청사진입니다.
도커 이미지를 사용하여 여러 개의 컨테이너를 생성할 수 있습니다.

이미지는 모든 설정 명령들과 코드가 포함된 공유 가능 패키지 입니다.
컨테이너는 이런 이미지의 구체적인 실행 인스턴스라고 할 수 있습니다.

Dockerhub에는 다른 사람들이 만든 이미지 혹은 공식 이미지를 다운로드 받을 수 있고,
Dockerfile을 사용하면 커스텀 이미지를 빌드할 수 있습니다.

```dockerfile
# FROM 명령어는 로컬 혹은 Dockerhub에 있는 베이스 이미지(baseImage)를 가져옵니다.
FROM node

COPY . /app

# Default working directory is `/`
WORKDIR /app

# RUN 명령어는 도커 이미지를 빌드할 떄 사용되는 명령어
RUN npm install

# EXPOSE 명령어는 현재 컨테이너가 해당 포트를 사용한다는 것을 도커에게 알린다.
# 이는 호스트 포트와 바인딩되는 것이 아니라 도커 내부에서만 노출하는 것이기 떄문에
# 일반적으로 도커 내부에서 컨테이너 간 통신하는 경우(inter-container communication) 혹은 문서화 용도로 사용된다.
# 실제로 호스트와 바인딩하려면 -p 옵션을 사용하여 호스트 포트와 컨테이너 포트를 매핑해야 합니다.
EXPOSE 80

# CMD 명령어는 도커 컨테이너를 실행할 때 사용되는 명령어
CMD ["node", "server.js"]
```

해당 Dockerfile을 사용하여 이미지를 빌드하기 위해서는 다음 명령어를 사용합니다.

```bash
docker build .
```

이미지를 빌드하면 이미지 ID가 생성되고, 해당 이미지 ID를 사용하여 컨테이너를 생성할 수 있습니다.

```bash
docker run -p 3000:80 <image_id>
```

도커 이미지는 불변하다는 특징이 있습니다. 이미지를 한 번 빌드하면 그 이미지는 변경되지 않습니다. 
따라서 만약 코드에 변경이 생기면 이미지를 다시 빌드해야 합니다.

### 레이어 기반 아키텍처

도커 이미지는 레이어를 기반 아키텍처를 사용합니다.
Dockerfile의 모든 명령은 이미지 레이어를 생성하고, 
이미지를 빌드할 때 변경된 부분의 명령과 그 이후의 모든 명령이 재평가 됩니다.

```bash
# file changed
 => CACHED [1/4] FROM docker.io/library/node@sha256:a8ba58f54e770a0f910ec  0.0s
 => [2/4] COPY . /app                                                      0.0s
 => [3/4] WORKDIR /app                                                     0.0s
 => [4/4] RUN npm install                                                  2.3s
```

위 빌드 결과를 보면 파일이 변경되었기 때문에 COPY 이후의 모든 명령어가 다시 실행되는 것을 볼 수 있습니다.

```bash
# file not changed
 => [1/4] FROM docker.io/library/node@sha256:a8ba58f54e770a0f910ec36d25f8  0.0s
 => CACHED [2/4] COPY . /app                                               0.0s
 => CACHED [3/4] WORKDIR /app                                              0.0s
 => CACHED [4/4] RUN npm install                                           0.0s
 => exporting to image                                                     0.0s
 ```

반면 파일이 변경되지 않은 경우에는 COPY 이후의 명령어가 캐싱되어 다시 실행되지 않습니다.

하지만 실제로 npm install 명령어가 필요한 경우는 오직 package.json 파일이 변경되었을 때 뿐입니다.
따라서 package.json 파일만 복사하고 npm install 명령어를 실행하면 불필요한 빌드 시간을 줄일 수 있습니다.

```bash
 => [1/5] FROM docker.io/library/node@sha256:a8ba58f54e770a0f910ec36d25f8  0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 1.14kB                                        0.0s
 => CACHED [2/5] WORKDIR /app                                              0.0s
 => CACHED [3/5] COPY package.json /app                                    0.0s
 => CACHED [4/5] RUN npm install                                           0.0s
 => [5/5] COPY . /app                                                      0.0s
 => exporting to image                                                     0.0s
 ```

위 결과를 보면 코드가 변경되었음에도 package.json 파일은 변경되지 않았기 때문에
npm install 명령어가 캐싱된 것을 볼 수 있습니다.

### 요약 정리

![docker_aws.png](images%2Fdocker_image.png)

컨테이너가 이미지의 코드와 환경을 새 컨테이너로 복사하거나 새 파일로 복사하지 않습니다.
컨테이너는 이미지에 저장된 환경을 사용합니다.
그리고 그 위에 부가적인 레이어(리소스, 메모리 등)을 추가합니다.
컨테이너는 이미지가 빌드되면 해당 이미지를 활용합니다.

위 그림에서 1개의 이미지와 2개의 컨테이너가 있지만
실제로 코드에 대한 복사는 이미지를 빌드할 떄 1번만 수행됩니다.
컨테이너를 생성할 때는 코드가 복사되는 것이 아니라
이미지를 활용합니다.

![docker_aws.png](images%2Fdocker_layered.png)

이미지의 내부 내용은 레이어드 아키텍처로 구성됩니다.
이미지의 모든 명령은 캐시 가능한 레이어를 생성하고,
레이어를 사용하여 이미지 재구축을 및 공유를 효율적으로 수행합니다.

위 그림에서 코드에 대한 변경이 발생해도,
`package.json`에서 의존성 추가나 변경이 발생하지 않으면
`npm install`은 캐싱되어 수행되지 않습니다.

## 도커 이미지와 컨테이너 관리

### 컨테이너 실행 및 중지

위에서 언급한 것처럼 이미지를 사용하여 컨테이너를 생성하고 실행할 수 있습니다.

```bash
docker run -p 3000:80 <image_id>
```

현재 실행중인 컨테이너를 확인하려면 다음 명령어를 사용합니다.

```bash
docker ps
```

컨테이너를 중지하려면 다음 명령어를 사용합니다.

```bash
docker stop <container_id>
```

중지된 컨테이너를 포함하는 모든 컨테이너를 확인하려면 `ps` 명령어에 `-a` 옵션을 사용합니다.

```bash
docker ps -a
```

`run` 명렁어는 컨테이너를 생성하고 실행하는 명령어이기 떄문에
실행할 때마다 새로운 컨테이너가 생성됩니다.

하지만 이미 생성된 컨테이너를 다시 실행하려면 `start` 명령어를 사용합니다.
`start` 명령어는 중지된 컨테이너를 다시 실행하는 명령어입니다.

```bash
docker start <container_id>
```

### 컨테이너 Attach 및 Detach

`docker run` 명령어를 사용하여 컨테이너를 실행하면 컨테이너가 foreground로 실행되면서 로그를 출력합니다.
반면에 `docker start` 명렁어를 사용하면 컨테이너가 background로 실행되며 로그를 출력하지 않습니다.

`run` 명령어는 기본적으로 `attach` 모드로 실행되며, `start` 명령어는 `detach` 모드로 실행됩니다.

attach 모드는 현재 실행하는 터미널에 컨테이너의 로그를 컨테이너의 로그를 출력하고,
detach 모드는 컨테이너를 백드라운드로 실행하여 로그를 출력하지 않고 터미널을 반환합니다.

docker run 명령어를 사용할 때 `-d` 옵션을 사용하면 `detach` 모드로 실행할 수 있습니다.

```bash
docker run -d -p 3000:80 <image_id>
```

반대로 docker start 명령어를 사용할 때 `-a` 옵션을 사용하면 `attach` 모드로 실행할 수 있습니다.

```bash
docker start -a <container_id>
```

또한 이미 실행중인 컨테이너에 연결하려면 `attach` 명령어를 사용합니다.

```bash
docker attach <container_id>
```

attach 모드를 사용하지 않고 로그를 확인하려면 `logs` 명령어를 사용합니다. 터미널은 반환됩니다.

```bash
docker logs <container_id>
```

이전에 실행한 컨테이너의 로그를 포함하여 attach 모드로 실행하려면 `-f` 옵션을 사용하여 `follow` 모드로 실행합니다.
이 경우 터미널을 반환하지 않습니다.

```bash
docker logs -f <container_id>
```

### 컨테이너 인터렉티브 모드

컨테이너를 attach 모드로 실행하면 출력값을 확인할 수 있지만, 입력값을 전달할 수 없습니다.

컨테이너에 입력값을 전달하기 위해서는 먼저 컨테이너를 생성할 때
`-t` 옵션을 사용하여 `pseudo-TTY`를 할당하고,
`-i` 옵션을 사용하여 `interactive` 모드로 실행해야 합니다.

```bash
docker run -it <image_id>
```

start 명령어도 마찬가지로 `-i` 옵션을 사용하여 `interactive` 모드로 실행할 수 있습니다.
하지만 해당 컨테이너가 생성될 때 반드시 tty를 할당한 상태여야 합니다.

```bash 
docker start -ia <container_id>
```

### 컨테이너 삭제

컨테이너를 삭제하려면 `rm` 명령어를 사용합니다.
만약 컨테이너가 실행 중이라면 `stop` 명령어를 사용하여 컨테이너를 중지한 후 삭제해야 합니다.

```bash
docker rm <container_id> <container_id> ...
```

컨테이너 이름을 사용하여 삭제할 수도 있습니다.

```bash
docker rm <container_name> <container_name> ...
```

이미지를 삭제하려면 `rmi` 명령어를 사용합니다.
만약 이미지를 사용하는 컨테이너가 있다면 해당 컨테이너를 삭제한 후 이미지를 삭제해야 합니다.

```bash
docker rmi <image_id> <image_id> ...
```
 
`prune` 명령어를 사용하여 사용하지 않는 이미지를 삭제할 수 있습니다.

```bash
docker image prune
```

### 중지된 컨테이너 자동 삭제

컨테이너를 생상할 때 `--rm` 옵션을 사용하면 컨테이너를 중지할 때 자동으로 삭제할 수 있습니다.

웹 어플리케이션의 경우 컨네이너를 중지하는 이유는 대부분 코드 변경으로 인해 새로 빌드하기 위해서 입니다.  
따라서 컨테이너를 중지하면 해당 컨테이너는 더 이상 사용하지 않는 것이므로 삭제하는 것이 좋습니다.

```bash
docker run -p 3000:80 -d --rm <image_id>
```

### 이미지 Inspect

`inspect` 명령어를 사용하여 이미지의 상세 정보를 확인할 수 있습니다.

```bash
docker image inspect <image_id>
```

### 컨테이너 파일 복사

`cp` 명령어를 사용하여 호스트의 파일을 컨테이너로 복사할 수 있습니다.

```bash
docker cp <host_file> <container_id>:<container_file>
```

반대로 컨테이너의 파일을 호스트로 복사할 수도 있습니다.

```bash
docker cp <container_id>:<container_file> <host_file>
```

### 이미지와 컨테이너 이름 및 태그 지정

컨테이너는 `--name` 옵션을 사용하여 이름을 지정할 수 있습니다.

```bash
docker run --name <container_name> <image_id>
```

이미지는 컨테이너와 달리 tag 속성을 갖고 있습니다.

이미지 태그는 repository과 tag로 구성되어 있고, repository와 tag를 합쳐서 고유 식별자로 사용합니다. 
repository는 이미지의 이름을 의미하고, tag는 이미지의 버전을 의미합니다.

예를 들어 `node:14`는 `node` 이미지의 `14` 버전을 의미합니다.

이미지에 -t 옵션을 사용하여 이름과 태그를 지정할 수 있습니다.

> 태그를 지정하지 않으면 `latest` 태그가 자동으로 지정됩니다.

```bash
docker build -t <repository>:<tag> .
```
