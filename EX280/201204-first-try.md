# 201204 初めての受講

## 結果

```
Exam domain number:     14
Passing score:          210
Your score:             56

Result: NO PASS

Performance on exam objectives:

  OBJECTIVE: SCORE
  Manage OpenShift Container Platform: 17%
  Manage users and policies: 50%
  Control access to resources: 43%
  Configure networking components: 0%
  Configure pod scheduling: 0%
```

全然ダメだった…

## 次回に向けた反省点

- OpenShift以外で大幅に時間を使ってしまった。

htpasswdコマンドをopenshiftへlogin後に使用できず、作成に時間がかかった

$ scp ~/ex280-htpasswd root@workspace.~.com:/root/

**止まったら、一回飛ばせばよかった。**


- マニュアルを全然活用できなかった。

時間に余裕があれば、マニュアルで確認できた

ex) LimitRange をyamlで書かせる問題があったが、マニュアル確認すればできたはず…


- 基本的に問題で使うように言われているpodとかは作成済み

- コンソールは使わない


## 問題(自分の記憶にある限り)

- htpasswd

- clusterrole

> kubeadminユーザ-の削除

https://access.redhat.com/documentation/ja-jp/openshift_container_platform/4.5/html/authentication_and_authorization/removing-kubeadmin_removing-kubeadmin


- role

- グループの作成

- quotaの作成

- limitrangeの作成

https://access.redhat.com/documentation/ja-jp/openshift_container_platform/4.5/html/nodes/nodes-cluster-limit-ranges

- route作成

- セキュアなroute作成

- トラブルシューティングが3問
