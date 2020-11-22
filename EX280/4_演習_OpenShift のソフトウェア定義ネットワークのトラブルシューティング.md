# トラブルシューティング(むずい)

## 目的

- ソフトウェア・デファインド・ネットワークをトラブルシューティングする
- クラスタネットワークの進入を制御する
- 外部ルートを作成して編集する & 自己署名証明書を作成する(エッジルート)
- TLS 証明書を使用してルートをセキュリティ保護する（パススルールート)

## めも

```
oc debug で、擬似アプリを起動できて、動作テストとかできる
oc status、oc logs、oc describe、oc execでエラー原因を探せるかなー
```



## 使ったコマンド

- 前提
```
私はOpenShift開発者として、To Do Node.js アプリケーションをOpenShift に移行済みです。
このアプリケーションは、データベース用とフロントエンド用の2 つのデプロイで構成されています。Pod 間の通信用に 2 つのサービスも含まれています。 

アプリケーションは初期化されているように見えますが、Web ブラウザーからアクセスすることができません。トラブルシューティングして問題を修正します。
```

> $ oc cp db-data.sql mysql-94dc6645b-hjjqb:/tmp/

ローカルにあるdb-data.sql というファイルを、mysql-94dc6645b-hjjqb という名のpodの/tmp配下にコピー

> $ oc rsh mysql-94dc6645b-hjjqb bash

podにリモートアクセス

> 1000570000@mysql-94dc6645b-hjjqb:/$ mysql -u root -p$MYSQL_ROOT_PASSWORD items < /tmp/db-data.sql

oc cpコマンドで持ってきたファイルを使用して、データベースを復元

---

> $ oc expose service frontend --hostname todo.apps.ocp4.example.com

http://todo.apps.ocp4.example.com でアクセスできるようにした

> $ oc logs frontend-57b8b445df-f56qh(pod名)

> $ oc debug -t deployment/frontend

**oc debug コマンドで、デバックモードのpodを起動できて、テストとかできる**

(公式)：イメージおよび設定の問題をデバッグする際、実行中の Pod 設定の正確なコピーを取得し、シェルでトラブルシュートを実行することができます。

疎通先のIPアドレスとPORTから接続確認テスト(curl)とかできる

> $ oc get svc frontend 
> $ oc get pod frontend

> $ oc edit svc/frontend