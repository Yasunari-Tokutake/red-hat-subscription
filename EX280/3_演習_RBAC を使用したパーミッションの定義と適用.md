## 目的

#### ロールベースのアクセス制御(RBAC)を定義する
#### ユーザーにアクセス許可を適用する
#### 機密情報を管理するためのシークレットを作成して適用する
#### セキュリティコンテキストの制約(SCC)を使用してサービスアカウントを作成し、アクセス許可を適用する

- メモ
```
> oc get xxは便利
> role名を調べたい時は、oc get clusterrole
> -o wideで、oc getでさらなる情報を得ることができる
```

##  使ってやつ

> $ oc get clusterrolebinding -o wide | grep self-provisioner

> $ oc describe clusterrolebindings self-provisioners (getより詳細に)

> $ oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth

２回目：×

> $ oc policy add-role-to-user admin leader (admin権限があるuserならadmコマンドは無くてもいい)

> $ oc adm groups new dev-group

> $ oc adm groups add-users dev-group developer

> $ oc get groups

> $ oc get clusterrole

> $ oc get rolebindings **-o wide**

> $ oc adm policy add-cluster-role-to-group --rolebinding-name self-provisioners self-provisioner system:authenticated:oauth

--rolebinding-name で名付けができる(11/24)
