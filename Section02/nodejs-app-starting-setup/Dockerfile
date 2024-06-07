# Dockerfile을 사용하면 커스텀 이미지를 빌드할 수 있습니다.

# FROM 명령어는 로컬 혹은 Dockerhub에 있는 베이스 이미지(baseImage)를 가져옵니다.
FROM node

# Default working directory is `/`
WORKDIR /app

# package.json 파일만 복사하여 npm install을 실행합니다.
# 다른 코드가 변경되었을 때 package.json 파일이 변경되지 않았다면
# 캐시된 이미지를 사용할 수 있기 때문에 빌드 시간을 단축할 수 있습니다.
# COPY . /app
COPY package.json /app

# RUN 명령어는 도커 이미지를 빌드할 떄 사용되는 명령어
RUN npm install

COPY . /app

# EXPOSE 명령어는 현재 컨테이너가 해당 포트를 사용한다는 것을 도커에게 알린다.
# 이는 호스트 포트와 바인딩되는 것이 아니라 도커 내부에서만 노출하는 것이기 떄문에
# 일반적으로 도커 내부에서 컨테이너 간 통신하는 경우(inter-container communication) 혹은 문서화 용도로 사용된다.
# 실제로 호스트와 바인딩하려면 -p 옵션을 사용하여 호스트 포트와 컨테이너 포트를 매핑해야 합니다.
EXPOSE 80

# CMD 명령어는 도커 컨테이너를 실행할 때 사용되는 명령어
CMD ["node", "server.js"]
