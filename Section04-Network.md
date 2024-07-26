# 도커 네트워크

- [도커 네트워크](#도커-네트워크)
  - [컨테이너에서 외부 네트워크로 통신하는 경우](#컨테이너에서-외부-네트워크로-통신하는-경우)
  - [컨테이너에서 호스트로 통신하는 경우](#컨테이너에서-호스트로-통신하는-경우)
  - [컨테이너에서 다른 컨테이너로 통신하는 경우](#컨테이너에서-다른-컨테이너로-통신하는-경우)

도커 컨테이너가 통신하는 경우는 다음과 같이 3가지 경우가 있습니다.

![docker_network_case.png](images%2Fdocker_network_case.png)

* 컨테이너에서 외부 네트워크로 통신하는 경우
* 컨테이너에서 호스트로 통신하는 경우
* 컨테이너에서 다른 컨테이너로 통신하는 경우

## 컨테이너에서 외부 네트워크로 통신하는 경우

컨테이너 내부 서버에서 axios 등을 사용하여 외부 네트워크로 통신하는 것은 별도의 설정을 하지 않아도 가능합니다.

```javascript
axios.get('https://jsonplaceholder.typicode.com/posts')
  .then((response) => {
    console.log(response.data);
  });
```

## 컨테이너에서 호스트로 통신하는 경우

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

## 컨테이너에서 다른 컨테이너로 통신하는 경우

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