# ch-02 docker-swarm-practice

## ゴール
2つのサービスをデプロイするところまで実施します。


## 2つめのAPPの作成

greetingAPPを作成します。  
内容はこのgreetingフォルダの中身を丸コピでいいです。
7999番ポートを使用します。


## デプロイする
chｰ01で実施した時と全く同じやり方です。
  
greetingフォルダ内で実行します。

```
docker-compose build
docker-compose push 
docker stack deploy --compose-file docker-compose.yml greeting
```

curlで確認してみましょう。

```
curl http://{IPアドレス}:7999
Hello
```
挨拶が帰ってきました。

続けて、stackdemoもデプロイしてみます

stackdemoフォルダ内で実行します。

```
docker-compose build
docker-compose push 
docker stack deploy --compose-file docker-compose.yml stackdemo
```

curlで確認してみましょう。

```
curl http://{IPアドレス}:8000
Hello World! I have been seen 1 times.
```

これで２つのサービスをデプロイ出来ました。
greetingAPPだけ変更してデプロイすることも可能です。  
まとめて上げなくていいので楽です。