# 도커 이미지와 컨테이너

- [도커 이미지와 컨테이너](#도커-이미지와-컨테이너)
  - [도커 이미지](#도커-이미지)
    - [도커 이미지란?](#도커-이미지란)
    - [레이어 기반 아키텍처](#레이어-기반-아키텍처)

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