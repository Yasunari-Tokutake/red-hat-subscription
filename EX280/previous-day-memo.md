
">" これは間違ったやつ

## 第3章

### OAuthの編集

$ oc get oauth **cluster** -o yaml > oauth.yaml

oauthの名前は、clusterか

> OAuth変更
```
spec: 
  identityProviders: 
  - name: myuser
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: myuser-secret
```

$ oc replace -f oauth.yaml

### HTPasswdファイルの作成

htpasswdでのsecretで使用するnamespaceはopenshift-config(oauthとの兼ね合い)

#### 新規

$ htpasswd -c -b ~/htpasswd tokutake Yasunari8930

~~$ oc create secret generic htpasswd-secret --from-file=~/htpasswd~~

> $ oc create secret generic htpasswd-secret --from-file **htpasswd**=~/htpasswd -n **openshift-config**

作成したseretをoauthに組み込む


#### 既存

$ oc extract secret/htpasswd-secret -n openshift-config --to=~/ --confirm

> $ oc set data secret/htpasswd-secret -n openshift-config --from-file htpasswd=~/htpasswd


### 削除

$ htpasswd -D ~/htpasswd tokutake Yasunari8930

$ oc set data secret/htpasswd-secret -n openshift-config --from-file htpasswd=~/htpasswd

$ oc delete user tokutake

$ oc delete identity myuser:tokutake


### 管理者権限を与える

$ oc adm policy add-cluster-role-to-user cluster-admin student



## 第4章

role、clusterroleは影響範囲を表す

ユーザーやグループに付加されているroleの確認

$ oc get rolebindings -o wide

$ oc get clusterrolebindings -o wide | grep self-provisioner

### secret

namespace 内で共有可能

$ oc create secret generic test -from-literal keys=xxxx --from-literal=yyyy

Pod のサービスアカウントを更新して、シークレットへの参照を許可します。

$ oc secrets add --for mount sa/test secret/test

- podへの環境変数

1. 
```
env:
  - name: MYSQL_ROOT_PASSWORD 
    valueFrom:
      secretKeyRef:
        name: demo-secret 
        key: root_password 
```

2. 

$ oc set env deployment/test --from=secret/test --prefix=MYSQL_

pod内にsecretをファイルとしてマウント

$ oc set volume deployment/test --add --type secret --secret-name test --mount-path /xxxx


環境変数のprefixはどうやれば検索？できる -> ~~以下のコマンドでprefixは分かる~~ 間違い

> $ oc set env pod/xxxxx-xyjsdhudbcdcw **--list**

oc create secret generic mysql --from-literal user=myuser --from-literal password=redhat123 --from-literal database=test_secrets --from-literal hostname=mysql 

**エラーには、oc describeもしくは、oc logs**


### scc

$ oc create sa gitlab-sa

$ oc adm policy add-sc-to-user anyuid -z gitlab-sa

> $ oc set sa deployment gitlab gitlab-sa  (△)

