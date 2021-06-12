# ch-01 docker-swarm-practice

以下の手順をより詳細に知りたい場合は、参考URLを参照  
参考URL：https://docs.docker.com/engine/swarm/stack-deploy/

## ゴール
２回デプロイしてレスポンスの内容を更新させるところまで実施します。

## swarmの作成
Dockerのコンテナーオーケストレーションツールswarmを使います。

```
docker swarm init --advertise-addr {IPアドレス}
```

## ローカルにregistoryを作成  
registoryを作成します  
デプロイする時に、ここからイメージを取ります

```
docker service create --name registry --publish published=5000,target=5000 registry:2
```

## APPの作成
stackdemoAPPを作成します  
このリポジトリにあるstackdemoフォルダをコピーしてOKです

## Build

ビルドを実施してイメージを作成します

```
cd stackdemo
docker-compose build
```

## registoryにpushする
先程作成したregisotoryにイメージをpushします

```
docker push
```

こんな感じのログが出ます

 ```
 Pushing web (127.0.0.1:5000/stackdemo:latest)...
The push refers to repository [127.0.0.1:5000/stackdemo]
31291e7b85f6: Layer already exists
aa476aa983e8: Layer already exists
62de8bcc470a: Layer already exists
58026b9b6bf1: Layer already exists
fbe16fc07f0d: Layer already exists
aabe8fddede5: Layer already exists
bcf2f368fe23: Layer already exists
latest: digest: sha256:cfb7ea36a09294f647352bf4f5310a0b201bdeefd721e0be9dd4f19ee0736708 size: 1786 
 ```
 
 ## デプロイ（1回目）
 デプロイを実行します  
 
 ```
 docker stack deploy --compose-file docker-compose.yml stackdemo
 ```
 
 デプロイ実行時にはこんな感じのログが出ます
 
 ```
Creating network stackdemo_default
Creating service stackdemo_web
Creating service stackdemo_redis
 ```
 
 補足：docker-compose.ymlファイルをみたら、この時にregisotryからイメージを取得していることが分かります。
 
 
 ```
 version: "3.0"

services:
  web:
    image: 127.0.0.1:5000/stackdemo <- コレ
    build: .
    ports:
      - "8000:8000"
  redis:
    image: redis:alpine
 ```
 
 curlを叩いて確認しましょう
 
```
curl http://{IPアドレス}:8000
Hello World! I have been seen 1 times.
```
 ## デプロイ（2回目）
 2回目のデプロイを実行して内容を更新してみます。  
 ビルド内容が変わらないのでdocker-buildとdocker-pushする意味はないのですが、普段CDでやってる作業を意識するためにやります。  
  
 せっかくなので、app.pyを修正します。
 app.pyの内容を以下のように変更しました（挨拶を消しました）。  
 
 ```
 from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return ' {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000, debug=True)
 ```
 
回数だけ帰ってくるようになった状態でデプロイしてみましょう。
先程と同じ手順を踏みます。
 
```
docker-compose build
docker-compose push 
docker stack deploy --compose-file docker-compose.yml stackdemo
```

ログです。updateされていることがわかります。

```
Updating service stackdemo_web (id: 3krs5qhxrus35r1p9jp6jmaqv)
Updating service stackdemo_redis (id: gvp9l3kgw7d1l28fusudvtydr)
```

curlを叩いて確認してみます

```
curl http://{IPアドレス}:8000
1 times.
```

回数だけ帰ってきました。
