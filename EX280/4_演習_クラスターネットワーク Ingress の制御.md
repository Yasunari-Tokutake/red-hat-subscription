# TLS 証明書によって保護されているアプリケーションを公開(むずい)

## 目的

- ソフトウェア・デファインド・ネットワークをトラブルシューティングする
- クラスタネットワークの進入を制御する
- 外部ルートを作成して編集する & 自己署名証明書を作成する(エッジルート)
- TLS 証明書を使用してルートをセキュリティ保護する（パススルールート)

## めも

```
1. 証明書には、2種類(ここでは)

    Openshifftによる証明書への署名 or opensslコマンドで作成

2. 安全なルート作成も、2種類

    edge or passthrough

3. 証明書の整合性の確認方法(curl)
```


## 使ったコマンド

```
アプリケーション開発者として、アプリケーションを OpenShift にデプロイする準備ができています。
このアクティビティーでは、アプリケーションの 2 つのバージョンをデプロイします。1 つは暗号化されていないトラフィック (HTTP) を介して公開され、もう 1 つは安全なトラフィックで公開されます。
```

#### 安全でない(http)route

> $ oc expose svc todo-http --hostname todo-http.apps.ocp4.example.com

---

## 証明書

#### Openshifftによる証明書への署名

特にsecretとかで指定しないサービスを外部公開する時、勝手になる


#### opensslコマンドで作成

Pod にシークレットとして割り当てる CA 署名証明書を生成し、アプリケーション作成までの流れ

> $ openssl genrsa -out training.key 2048

training.key という秘密鍵を作成

> $ openssl req -new -subj "/C=US/ST=North Carolina/L=Raleigh/O=Red Hat/CN=todo-https.apps.ocp4.example.com" -key training.key -out training.csr

training.csr という証明書要求を作成

> $ openssl x509 -req -in training.csr -passin file:passphrase.txt -CA training-CA.pem -CAkey training-CA.key -CAcreateserial -out training.crt -days 1825 -sha256 -extfile training.ext

training.crt という署名済み証明書を作成

> $ oc create secret tls todo-certs --cert certs/training.crt --key certs/training.key

作成した秘密鍵と証明書を埋め込んだtls シークレットを作成

> $ oc create -f todo-app-v2.yaml

secretにtodo-sertsが指定されたyamlファイルからアプリケーションを作成している 



## 安全なルート

**oc createで作成、oc exposeではない**

#### edge route

> $ oc create route edge todo-https --service todo-http --hostname todo-https.apps.ocp4.example.com

svc: todo-httpをedge routeで外部公開

#### passthrough route

> $ oc create route passthrough todo-https --service todo-https --port 8443 --hostname todo-https.apps.ocp4.example.com



## 証明書の整合性の確認方法(curl)

#### 「Openshifftによる証明書への署名」の場合

- **OpenShift による証明書の署名方法を確認する方法の1 つは、Ingress Operator が使用する CA を取得することです。これにより、CA と照合してエッジ証明書を検証できる。**

管理者ユーザーで、

> $ oc extract secrets/router-ca --keys **tls.crt** -n openshift-ingress-operator

namespace: openshift-ingress-operatorからsecrets/router-caのkeyのみを抽出

> $ curl -I -v --cacert tls.crt https://todo-https.apps.ocp4.example.com
    > -I, -v : 指定でヘッダーあたりの情報のみ出力
    > --cacert : 指定した証明書ファイルを使用してピアを検証するようcurlに指示

```
...output omitted...
* Server certificate:
*  subject: CN=*.apps.ocp4.example.com
*  start date: Jul 29 18:35:37 2020 GMT
*  expire date: Jul 29 18:35:38 2022 GMT
*  subjectAltName: host "todo-https.apps.ocp4.example.com" matched cert's "*.apps.ocp4.example.com"
*  issuer: CN=ingress-operator@1596047735
*  SSL certificate verify ok.                <- これが検証結果
...output omitted...
```

#### 「opensslコマンドで作成」の場合

> $ curl -I -v --cacert certs/training-CA.pem https://todo-https.apps.ocp4.example.com

```
...output omitted...
* Server certificate:
*  subject: C=US; ST=North Carolina; L=Raleigh; O=Red Hat; CN=todo-https.apps.opc4.example.com
*  start date: Aug  3 17:40:30 2020 GMT
*  expire date: Aug  2 17:40:30 2025 GMT
*  subjectAltName: host "todo-https.apps.ocp4.example.com" matched cert's "*.apps.ocp4.example.com"
*  issuer: C=US; ST=North Carolina; L=Raleigh; O=Red Hat; CN=ocp4.example.com
*  SSL certificate verify ok.               <- これ
...output omitted...
```



