# 도커 네트워크

- [도커 네트워크](#도커-네트워크)
  - [도커 통신](#도커-통신)
    - [컨테이너에서 외부 네트워크로 통신하는 경우](#컨테이너에서-외부-네트워크로-통신하는-경우)
    - [컨테이너에서 호스트로 통신하는 경우](#컨테이너에서-호스트로-통신하는-경우)
    - [컨테이너에서 다른 컨테이너로 통신하는 경우](#컨테이너에서-다른-컨테이너로-통신하는-경우)
  - [도커 네트워크](#도커-네트워크-1)
  - [정리](#정리)

## 도커 통신

도커 컨테이너가 통신하는 경우는 다음과 같이 3가지 경우가 있습니다.

![docker_network_case.png](images%2Fdocker_network_case.png)

* 컨테이너에서 외부 네트워크로 통신하는 경우
* 컨테이너에서 호스트로 통신하는 경우
* 컨테이너에서 다른 컨테이너로 통신하는 경우

### 컨테이너에서 외부 네트워크로 통신하는 경우

컨테이너 내부 서버에서 axios 등을 사용하여 외부 네트워크로 통신하는 것은 별도의 설정을 하지 않아도 가능합니다.

```javascript
axios.get('https://jsonplaceholder.typicode.com/posts')
  .then((response) => {
    console.log(response.data);
  });
```

### 컨테이너에서 호스트로 통신하는 경우

```javascript
mongoose.connect(
  'mongodb://host.docker.internal:27017/swfavorites',
  { useNewUrlParser: true },
  (err) => {
    if (err) {
      console.log(err);
    } else {
      app.listen(3000);
    }
  }
);
```

컨테이너에서 호스트로 통신하기 위해서는 도커에서 제공하는 `host.docker.internal`을 사용합니다.

### 컨테이너에서 다른 컨테이너로 통신하는 경우

컨테이너 간 통신을 하기 위해서는 내부 네트워크 IP를 사용할 수 있습니다.

```bash
$ docker run -d --name mongodb mongo
$ docker container inspect mongo
```

예를 들어 mongoDB를 컨테이너로 실행하고 container inspect를 통해 IP를 확인합니다.

```json
[
    {
        // ...
        "NetworkSettings": {
            // ...
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",  // HERE!
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    // ...
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2", // AND HERE!
                    // ...
                }
            }
        }
    }
]

```

위와 같이 IP를 확인하고 해당 IP로 접속하면 됩니다.

```javascript
mongoose.connect(
  'mongodb://172.17.0.2:27017/swfavorites',
  { useNewUrlParser: true },
  (err) => {
    if (err) {
      console.log(err);
    } else {
      app.listen(3000);
    }
  }
);
```

## 도커 네트워크

도커 네트워크를 사용하면 `컨테이너 간 통신`을 쉽게 할 수 있습니다.

도커 네트워크는 볼륨과 다르게 자동으로 생성하지 않습니다.

```bash
$ docker network create favorite-net
```

다음 명령어를 사용해서 현재 생성된 네트워크를 확인할 수 있습니다.

```bash
$ docker network ls
```

위와 같이 네트워크를 생성하고 컨테이너를 실행할 때 `--network` 옵션을 사용하여 네트워크를 지정할 수 있습니다.

```bash
$ docker run -d --name mongodb --network favorite-net mongo
```

같은 네트워크에 있는 컨테이너끼리는 `컨테이너 이름을 사용`하여 통신할 수 있습니다.

```java
mongoose.connect(
  'mongodb://mongodb:27017/swfavorites',
  { useNewUrlParser: true },
  (err) => {
    if (err) {
      console.log(err);
    } else {
      app.listen(3000);
    }
  }
);
```

또한 도커 네트워크 안에서는 모든 컨테이너가 자유롭게 통신할 수 있기 때문에 -p 옵션을 사용해서 포트를 열어주지 않아도 됩니다. 

## 정리

![docker_network_summary.png](images%2Fdocker_network_summary.png)

정리하면 다음과 같습니다.
* 외부 네트워크로 통신하는 경우
  * 별도의 설정이 필요 없음
* 호스트로 통신하는 경우
  * `host.docker.internal` 사용
* 다른 컨테이너로 통신하는 경우
  * `컨테이너 IP` 사용
  * 도커 네트워크 사용
    * `컨테이너 이름` 사용
    * `docker network create` 명령어로 네트워크 생성
    * `--network` 옵션 사용해서 컨테이너 실행