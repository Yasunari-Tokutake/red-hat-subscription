## 目的

#### ロールベースのアクセス制御(RBAC)を定義する
#### ユーザーにアクセス許可を適用する
#### 機密情報を管理するためのシークレットを作成して適用する
#### セキュリティコンテキストの制約(SCC)を使用してサービスアカウントを作成し、アクセス許可を適用する

- メモ
```
> SCCはPodに対して付与するロール！！(分かりやすい)

参考：https://qiita.com/kimura-y/items/7ac0ef6f08c33683ab98

> このPodはどのユーザーなら使っていいとか、使っちゃダメとか指定できる

    ex)
        anyuid: どのユーザーでも良い

> 順序
    > 作成したServiceAccount(sa)にSecurity Context Constraints(scc)を割り当てる -> podにsaを割り当てる
```


##  使ってやつ

> $ oc get scc (describeも可)

> $ oc create sa gitlab-sa

サービスアカウント作成

> $ oc adm policy add-scc-to-user anyuid -z gitlab-sa

そのサービスアカウントにanyuidを付与

**--help見ればわかるけど、-zでsaをユーザーとして指定できる**

> $ oc set serviceaccount deployment gitlab gitlab-sa

gitlabというpod?deployment？にsaを割り当てた -> このpodには誰でもアクセスできるようになった

> $ oc expose service gitlab --port 80