# ch-03 docker-swarm-practice

## ゴール
versionを切り替えてデプロイしてみます  
version1.0の作成とデプロイ -> version1.1の作成とデプロイ -> version1.0ヘ切り替え -> version1.1ヘ切り替え  
  
## version 1.0

versionを付与します

docker-compose.yml
```
version: "3.0"

services:
  web:
    image: 127.0.0.1:5000/stackdemo:1.0 <-ココ
    build: .
    ports:
      - "8000:8000"
  redis:
    image: redis:alpine
```
DockerfileからRUN pip install -r requirements.txtを消しました(わざと不具合を発生させます)
version1.0では、この状態でデプロイしたいと思います

Dockerfile
```
# syntax=docker/dockerfile:1
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
CMD ["python", "app.py"]

```

buildします
```
docker-compose build
```
  
docker imagesで確認してみます  
TAGに1.0がついていることが確認できます

```
REPOSITORY                   TAG          IMAGE ID       CREATED              SIZE
127.0.0.1:5000/stackdemo     1.0          fe457d32690b   7 seconds ago   72.9MB
```

1.0をデプロイしましょう

```
docker-compose push
sudo docker stack deploy --compose-file docker-compose.yml stackdemo
```

curlを叩いても何も帰ってきません  
失敗しています

```
curl http://{IPアドレス}:8000
Failed to connect to ...
```

## version 1.1

修正版のversion1.1を作ってデプロイしてみましょう

docker-compose.yml
```
version: "3.0"

services:
  web:
    image: 127.0.0.1:5000/stackdemo:1.1
    build: .
    ports:
      - "8000:8000"
  redis:
    image: redis:alpine
```

RUN pip install -r requirements.txtを復活させます

Dockerfile
```
# syntax=docker/dockerfile:1
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

ビルドします
```
docker-compose build
```

docker imagesで確認してみます  
TAGに1.1がついていることが確認できます

```
REPOSITORY                   TAG          IMAGE ID       CREATED              SIZE
127.0.0.1:5000/stackdemo     1.1          fe457d32690b   7 seconds ago   72.9MB
```

1.1をデプロイしましょう

```
docker-compose push
sudo docker stack deploy --compose-file docker-compose.yml stackdemo
```

```
Pushing web (127.0.0.1:5000/stackdemo:1.1)...
The push refers to repository [127.0.0.1:5000/stackdemo]
0b8f521793d0: Pushed
d28945aa85dc: Pushed
62de8bcc470a: Layer already exists
58026b9b6bf1: Layer already exists
fbe16fc07f0d: Layer already exists
aabe8fddede5: Layer already exists
bcf2f368fe23: Layer already exists
1.1: digest: sha256:4ae444e612be1ed848bf0789ff39062653ac9562c3224cbd37e8d093866a312e size: 1786
```

curlを使って復活したか確認してみます

```
curl http://{IPアドレス}:8000
Hello World! I have been seen 1 times.
```

どうやら復活したようです


## version切り替え

好奇心で不具合ありのバージョンに戻してみます


```
version: "3.0"

services:
  web:
    image: 127.0.0.1:5000/stackdemo:1.0
    build: .
    ports:
      - "8000:8000"
  redis:
    image: redis:alpine
```

buildやpushはいりません。1.0に関しては、もう実施したので。
デプロイします。

```
sudo docker stack deploy --compose-file docker-compose.yml stackdemo
```


curlを叩いてみましょう。。。  

```
curl http://{IPアドレス}:8000
Failed to connect to ...
```

ああ！何ということでしょう。  
バグありバージョン戻ってしまいました！

確認できたらversion1.1にして戻しておきます。

```
version: "3.0"

services:
  web:
    image: 127.0.0.1:5000/stackdemo:1.1
    build: .
    ports:
      - "8000:8000"
  redis:
    image: redis:alpine
```

docker-compose.ymlを書き換えたら急いでデプロイします
```
sudo docker stack deploy --compose-file docker-compose.yml stackdemo
``

curlを叩いて直ったかどうか確認してみます。
```
curl http://{IPアドレス}:8000
Hello World! I have been seen 2 times.
```

よかった。どうやら直っているようです。  