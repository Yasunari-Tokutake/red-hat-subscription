# Openshift Container Platformの管理

- DO180の6章 ~
- DO280の2章

- もしくは、マニュアルから必要そうなコマンドを抜き出す。

## コマンドライン・インタフェース(CLI)を使用して、OpenShift クラスタを管理および構成する

DO180の6,8章は確認しといた方がいいかな〜(ocコマンド)


---

- https://rol.redhat.com/rol/app/courses/do280-4.5/pages/ch02s03

## Web コンソールを使用して、OpenShift クラスタを管理および構成する

## プロジェクトを作成して削除する

oc new-project xxxx

oc new-app --name xxxx 

oc create -f xxxx.yaml --save-config

oc delete project xxxx


## Kubernetes リソースをインポート、エクスポート、設定する



## リソースとクラスタのステータスを確認する

oc status

## ログを確認する

oc logs

## クラスタイベントとアラートを監視する

## 一般的なクラスタイベントとアラートのトラブルシューティング

## 製品マニュアルを使用する

- [CLI リファレンス](https://access.redhat.com/documentation/ja-jp/openshift_container_platform/4.1/html-single/cli_reference/index)


oc exec -it xxxx /bin/bash