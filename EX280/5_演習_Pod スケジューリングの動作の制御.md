# Pod スケジューリングの動作の制御

## 目的

- リソース使用量を制限する
- 増加する要求に合わせてアプリケーションを拡張する　<- これ
- クラスタノードへの Pod 配置を制御する　<- これ


## めも

```
nodesについて
    podは必ずnodeの中で動く
    https://kubernetes.io/ja/docs/tutorials/kubernetes-basics/explore/explore-intro/

oc edit で、depoloymentを編集できて、nodeselectorを追加/変更でenvを指定
```


## 使ったコマンド

前提：helloというアプリケーション稼働中

> $ oc scale --replicas 4 deployment/hello

> $ oc get pods -o wide

```
NAME                     READY   STATUS    ...   IP           NODE       ...
hello-6c4984d949-78qsp   1/1     Running   ...   10.9.0.30    master02   ...
hello-6c4984d949-cf6tb   1/1     Running   ...   10.10.0.20   master01   ...
hello-6c4984d949-kwgbg   1/1     Running   ...   10.8.0.38    master03   ...
hello-6c4984d949-mb8z7   1/1     Running   ...   10.10.0.19   master01   ...
```

3つのnodeに分散している。 **この分散を制御したい**

アプリケーションワークロードが開発ノードまたは実稼働ノードに分散されるように、今回は envラベル を割り当ててノードを準備します。(ラベル名はなんでもいい、自分で把握していれば)

master01 ノードにenv=dev ラベルを割り当て、master02 ノードにenv=prod ラベルを割り当てます。 

> $ oc get nodes -L env

> $ oc label node master01 env=dev

> $ oc edit deployment/hello で env=dev を指定

```
...output omitted...
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      nodeSelector:                                      <- これを追記/変更
        env: dev                                         <-
      restartPolicy: Always
...output omitted...
```

