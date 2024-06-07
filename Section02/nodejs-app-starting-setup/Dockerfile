# Dockerfile을 사용하면 커스텀 이미지를 빌드할 수 있습니다.

# FROM 명령어는 로컬 혹은 Dockerhub에 있는 베이스 이미지(baseImage)를 가져옵니다.
FROM node

# Default working directory is `/`
WORKDIR /app

COPY . /app

# RUN 명령어는 도커 이미지를 빌드할 떄 사용되는 명령어
RUN npm install

# EXPOSE 명령어는 현재 컨테이너가 해당 포트를 사용한다는 것을 도커에게 알린다.
# 이는 호스트 포트와 바인딩되는 것이 아니라 도커 내부에서만 노출하는 것이기 떄문에
# 일반적으로 도커 내부에서 컨테이너 간 통신하는 경우(inter-container communication) 혹은 문서화 용도로 사용된다.
# 실제로 호스트와 바인딩하려면 -p 옵션을 사용하여 호스트 포트와 컨테이너 포트를 매핑해야 합니다.
EXPOSE 80

# CMD 명령어는 도커 컨테이너를 실행할 때 사용되는 명령어
CMD ["node", "server.js"]
