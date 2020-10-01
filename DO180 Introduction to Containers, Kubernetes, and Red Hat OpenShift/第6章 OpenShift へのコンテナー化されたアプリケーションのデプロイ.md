# 第6章 OpenShift へのコンテナー化されたアプリケーションのデプロイ

## Kubernetes および OpenShift アーキテクチャの説明

#### Kubernetes と OpenShift

**Kubernetes** はコンテナー化されたアプリケーションのデプロイメント、管理、およびスケーリングを簡潔化するオーケストレーションサービスである。

Kubernetes を使用する主な利点の 1 つは、複数のノードを使用して管理対象アプリケーションの復元力とスケーラビリティーを確保していることです。Kubernetes はコンテナーを実行する一連のノードサーバーのクラスターを形成し、複数のマスターサーバーによって集中管理されています。

**Red Hat OpenShift Container Platform** は Red Hat CoreOS と Kubernetes 上に構築されたモジュール式のコンポーネントとサービスのセットです。

RHOCP は遠隔管理、高度なセキュリティー、監視と監査、アプリケーションライフサイクル管理、開発者向けのセルフサービスインターフェースなどの PaaS 機能も備えています。

#### Kubernetes のリソースタイプの説明

- Pod (po)

IP アドレスや永続的ストレージボリュームなどのリソースを共有するコンテナーの集合です。Pod は Kubernetes の基本的な作業単位です。 

- サービス (svc)

Pod プールへアクセスするための 1 つの IP とポートの組み合わせを定義します。デフォルトでは、サービスにおけるクライアントから Pod への接続はラウンドロビン形式で実施されます。 

- レプリケーションコントローラー (rc)

Pod を異なるノードに複製 (水平方向にスケーリング) する方法を定義する Kubernetes リソース。レプリケーションコントローラーは、Pod とコンテナーに高可用性を提供するための基本的な Kubernetes サービスです。 

- 永続ボリューム (pv)

Kubernetes Pod が使用するストレージエリアを定義します。 

- 永続ボリューム要求 (pvc)

Pod によるストレージの要求です。PVC は、通常コンテナーのファイルシステムにストレージをマウントすることによって、そのコンテナーが PV を利用できるように PV を Pod にリンクします。 

- ConfigMaps (cm) と Secrets

他のリソースが使用できるキーと値のセットが含まれています。ConfigMaps と Secrets は通常、複数のリソースで使用される設定値を一元管理するために使用されます。Secret は ConfigMap のマップと異なり、Secret の値は常にエンコードされ (暗号化されていない)、そのアクセス権は少数の認証されたユーザーに制限されています。 


#### OpenShift のリソースタイプ

openshiftによって、kubernetesに追加される主なリソース

- デプロイ設定 (dc)

Pod に含まれるコンテナーのセット、および使用されるデプロイ戦略を表します。dc は基本的であるが拡張可能な継続的提供ワークフローも提供します。 

- ビルド設定 (bc)

OpenShift プロジェクトで実行されるプロセスを定義します。Git リポジトリに保存されているアプリケーションソースコードからコンテナーイメージをビルドするために、OpenShift Source-to-Image (S2I) 機能によって使用されます。bc は dc と連携して基本的であるが拡張可能な継続的インテグレーションワークフローおよび継続的デリバリーワークフローを提供します。 

- ルート

OpenShift ルーターによって認識される DNS ホスト名であり、アプリケーションとマイクロサービスのエントリーポイントになります。 


#### ネットワーキング

Kubernetes クラスターにデプロイされる各コンテナーは内部ネットワークから割り当てられた IP アドレスを持ち、この IP アドレスはコンテナーを実行しているノードからのみアクセスできます。コンテナの一時的な性質により、IP アドレスは割り当てと解放を常に繰り返しています。

Kubernetes はソフトウェア定義ネットワーク (SDN) を提供します。SDN では複数のノードから成る内部コンテナーネットワークが生成され、任意のポッドの任意のホスト内のコンテナーが他のホストのポッドにアクセスできます。SDN へのアクセスは同一の Kubernetes クラスタからのみ可能です。 

**OpenShift ではルートリソースを定義することによって、コンテナーへの外部アクセスがスケーラブルかつ簡単になります。**

ルートは、サービスの外部向け DNS 名とポートを定義します。ルーター (入力コントローラー) は、HTTP および TLS 要求を Kubernetes SDN 内のサービスアドレスに転送します。唯一の要件は、該当 DNS 名が RHOCP ルーターノードの IP アドレスにマッピングされることです。 



## Kubernetes リソースの作成

#### Red Hat OpenShift Container Platform (RHOCP) コマンドラインツール

RHOCP クラスターと対話するには、主に oc コマンドを使用します。
```
$> oc <command>
```

クラスターと対話するには、ほとんどのオペレーションでログインしたユーザーが必要となります。
```
$> oc login <clusterUrl>
```

#### ポッドのリソース定義構文の説明

RHOCP は、コンテナーを Kubernetes ポッド内で実行します。コンテナーイメージからポッドを作成するために、OpenShift はポッドのリソース定義を必要とします。

JSON または YAML 形式のテキストファイルとして記述しますが、oc new-app コマンドまたは OpenShift Web コンソールから、デフォルト値を記述したファイルを生成することも可能です。 

ポッドは、コンテナーおよびその他のリソースの集まりです。

ex) YAML 形式による WildFly アプリケーションサーバーのポッド定義
```
apiVersion: v1
kind: Pod (Kubernetes ポッドのリソースタイプを宣言)
metadata:
  name: wildfly (Kubernetes にあるポッドの一意の名前。これにより管理者はコマンドを実行可)
  labels:
    name: wildfly (Kubernetes の他のリソース (通常はサービス) による検出に使用できる name というキー付きのラベルを作成 )
spec:
  containers:
    - resources:
        limits :
          cpu: 0.5
      image: do276/todojee
      name: wildfly
      ports:
        - containerPort: 8080 (コンテナーに依存した属性。コンテナーのどのポートを公開するかを特定)
          name: wildfly
      env: (環境変数の集合を定義)
        - name: MYSQL_ENV_MYSQL_DATABASE
          value: items
        - name: MYSQL_ENV_MYSQL_USER
          value: user1
        - name: MYSQL_ENV_MYSQL_PASSWORD
          value: mypa55
```

#### サービスのリソース定義構文の説明

Kubernetes は、異なるワーカーのポッド同士が接続することを可能にする、仮想ネットワークを提供しています。

ただし、**Kubernetes において、ポッドが他のポッドの IP アドレスを検出することは容易ではありません。**

あるポッドのコンテナから、他のポッドのコンテナにネットワーク接続できるようにする働きがあります。ポッドはさまざまな原因で再起動し、毎回、前回とは異なる内部 IP アドレスが付与されます。再起動のたびに一方のポッドが他方のポッドの IP アドレスを検出するのではなく、**サービスが他方のポッドに静的 IP アドレスを用意していれば、再起動後、どのワーカーノードでポッドが動作しても問題は生じません。**

実世界のアプリケーションの多くは、単一のポッドとしては動作しません。ユーザーの要求が増えて水平拡張の必要が生じると、同じポッドのリソース定義に基づき、**多くのポッド上で同じコンテナーを実行するようになります。** サービスは一連のポッドを束ねて単一の IP アドレスを割り当て、クライアントから要求があると、**各ポッドに負荷を分散します。** 

サービスの向こう側で動作する**ポッド群を管理するのが DeploymentConfig リソース** です。DeploymentConfig リソースによって ReplicationController が埋め込まれて、ポッドのコピー (レプリカ)をいくつ作成するかが管理され、失敗した場合は新たに作成されます。

ex)JSON 構文の最小限のサービス定義
```
{
    "kind": "Service", (Kubernetes リソースの種類)
    "apiVersion": "v1",
    "metadata": {
        "name": "quotedb" (サービスの一意の名前)
    },
    "spec": {
        "ports": [ (クライアントはサービスポートに接続し、サービスはポッドの targetPort にパケットをフォワードする。)
            {
                "port": 3306, (port 属性はサービスによって公開されるポート)
                "targetPort": 3306 (argetPort 属性は、ポッドのコンテナー定義の containerPort 属性に対応する。)
            }
        ],
        "selector": {
            "name": "mysqldb" (パケットの転送先ポッドをサービスが見つける方法を示す。転送先ポッドのメタデータ属性には、これに対応するラベルがなければならない。ラベルが合致するポッドが複数見つかった場合、サービスはそれぞれに負荷を分散する。)
        }
    }
}
```

#### サービスの検出

アプリケーションは通常、環境変数を介して、サービスの IP アドレスやポートを調べます。OpenShift プロジェクト内の各サービスについて、次の環境変数が自動的に、同じプロジェクト内のポッドすべてに対して定義されます。

- SVC_NAME_SERVICE_HOST : サービスの IP アドレス。
- SVC_NAME_SERVICE_PORT : サービスの TCP ポート。 


OpenShift 内部の (ポッドからしか見えない) DNS サーバーを使って、ポッドからサービスを検出することもできます。各サービスには、次のような形式の FQDN を記述した SRV レコードが動的に割り当てられます。 

```
SVC_NAME.PROJECT_NAME.svc.cluster.local
```

環境変数を使ってサービスを検出する場合、ポッドを生成および起動するのは、サービスの作成後でなければなりません。一方、DNS クエリーでサービスを検出するようアプリケーションを実装していれば、Pod の起動後に作成されたサービスを検出できます。 

- アプリケーションが OpenShift クラスターの外からサービスにアクセスする2 つの方法

1. NodePort: これは旧式の Kubernetes ベースのアプローチです。サービスは、ワーカーノードホスト上の利用可能なポートにバインドすることにより外部クライアントに公開され、サービス IP アドレスへの接続をプロキシーします。oc edit svc コマンドを使用してサービス属性を編集し、type の値として NodePort を指定し、nodePort 属性のポート値を提供します。 OpenShift は次に、ワーカーノードホストのパブリック IP アドレスと nodePort で設定されたポート値を介してサービスへ接続をプロキシーします。

2. OpenShift ルート: これは、一意の URL を使ってサービスを公開する際に、**OpenShift で推奨されている方法です。oc expose コマンドを使用して外部アクセスのサービスを公開するか、 OpenShift Web コンソールからサービスを公開します。**


OpenShift には**ローカルポートをポッドポートに転送する oc port-forward コマンドが用意されています。** これはサービスリソースからのポッドへのアクセスとは異なります。 

ex) 開発者のマシンのポート 3306 を db ポッドのポート 3306 に転送し、このポートで MySQL サーバー (コンテナー内) によってネットワークアクセスが受け付けられます。 
```
[student@workstation ~]$ oc port-forward mysql-openshift-1-glqrp 3306:3306

注：このコマンドを実行するときはターミナルウィンドウを実行したままにしてください。ウィンドウを閉じるか、またはプロセスをキャンセルすると、ポートマッピングが停止します。 
```

#### 新規アプリケーションの作成

単純なアプリケーション、複雑な多層アプリケーション、マイクロサービスアプリケーションを、単一のリソース定義ファイルに記述できます。この単一のファイルに、多くの Pod 定義を収容します。さらに、Pod に接続するサービスの定義、アプリケーション Pod を水平拡張するためのレプリケーションコントローラーや DeploymentConfigs、アプリケーションデータを永続化する PersistentVolumeClaims など、OpenShift で管理可能な他のものも含めて、必要に応じて収容できます。 

**oc new-app コマンドに -o json または -o yaml オプションを指定して使用すると、スケルトンリソース定義ファイルを、それぞれ JSON または YAML 形式で作成できます。このファイルをカスタマイズして、oc create -f <filename> コマンドを使用してアプリケーションを作成するのに使用することや、他のリソース定義ファイルとマージしてコンポジットアプリケーションを作成することができます。**

oc new-app コマンドでアプリケーションポッドを作成すると、OpenShift 上でさまざまな方法によって実行することができます。**ポッドは、既存の Docker イメージからも、Dockerfile からも、また Source-to-Image (S2I) プロセスを使用すれば raw ソースコードからも作成できます。**

ex) Docker Hub の db=mysql に設定されたラベルを持つ mysql というイメージに基づいてアプリケーションを作成します。 
```
[student@workstation ~]$ oc new-app mysql --as-deployment-config MYSQL_USER=user MYSQL_PASSWORD=pass MYSQL_DATABASE=testdb -l db=mysql
```

ex) プライベート Docker イメージレジストリーにあるイメージに基づいてアプリケーションを作成します。 
```
oc new-app --docker-image=myregistry.com/mycompany/myapp --name=myapp --as-deployment-config
```

ex) Git リポジトリに格納されているソースコードに基づいてアプリケーションを作成します。 
```
oc new-app https://github.com/openshift/ruby-hello-world --name=ruby-hello --as-deployment-config
```

#### コマンドラインでの OpenShift リソースの管理

- oc get 

クラスター内のリソースに関する情報を取得することができます。一般に、このコマンドはリソースの最も重要な特性のみを出力し、詳細は省きます。 

ex) oc get RESOURCE_TYPE コマンドは、指定したタイプのすべてのリソースの概要を表示
```
NAME               READY   STATUS      RESTARTS  AGE
nginx-1-5r583      1/1     Running     0         1h
myapp-1-l44m7      1/1     Running     0         1h
```

- oc get RESOURCE_TYPE RESOURCE_NAME -o yaml

リソースの定義をエクスポートします。**バックアップを作成する、定義を修正する際に参照する、などの使い方が考えられます。** -o yaml オプションは YAML 形式でオブジェクト表現を出力しますが、これは -o json オプションを指定して JSON 形式に変更できます。 

- oc describe

oc get で取得される要約情報では不十分な場合は、oc describe コマンドを使用して追加情報を取得します。

ex)
```
Name:       mysql-openshift-1-glqrp
Namespace:          mysql-openshift
Priority:           0
PriorityClassName:  none
Node:               cluster-worker-1/172.25.250.52
Start Time:         Fri, 15 Feb 2019 02:14:34 +0000
Labels:             app=mysql-openshift
                    deployment=mysql-openshift-1
                    deploymentconfig=mysql-openshift
Annotations:        openshift.io/deployment-config.latest-version: 1
                    openshift.io/deployment-config.name: mysql-openshift
                    openshift.io/deployment.name: mysql-openshift-1
                    openshift.io/generated-by: OpenShiftNewApp
                    openshift.io/scc: restricted
Status:             Running
IP:                 10.129.0.85
```

- oc create

リソース定義に基づきリソースを作成します。通常、 oc get RESOURCE_TYPE RESOURCE_NAME -o yaml コマンドでリソース定義ファイルの雛形を作り、編集して使います。 

- oc edit

リソース定義に基づきリソースを編集します。デフォルトでは、このコマンドがリソース定義の編集向けに vi バッファーを開きます。 

- oc delete 

OpenShift クラスターからリソースを削除

ポッドなどの管理対象リソースを削除すると、同じリソースの新しいインスタンスが自動で作成されてしまうからです。プロジェクトが削除されると、このコマンドがそこに含まれるリソースやアプリケーションをすべて削除します。 そのため、OpenShift のアーキテクチャーに関する基本的な知識が必要。

- oc exec CONTAINER_ID options 

コンテナー内部でコマンドを実行します。このコマンドは、対話的に実行できるだけでなく、非対話的なバッチコマンドでスクリプトの一部としても実行できます。 


#### リソースのラベリング

同じプロジェクト内で多数のリソースを扱う場合は、それらのリソースをアプリケーション、環境、またはその他の基準で**グループ化**すると便利な場合があります。

これらのグループを確立するには、プロジェクト内のリソースのラベルを定義します。ラベルはリソースのメタデータセクションの一部であり、次のようにキー値ペアとして定義されています。 

ex)
```
apiVersion: v1
kind: Service
metadata:
...contents omitted...
  labels:
    app: nexus
    template: nexus-persistent-template
  name: nexus
...contents omitted...
```

oc サブコマンドの多くは、ラベル仕様からリソースを処理する **-l オプション**をサポートしています。**oc get コマンドの場合、-l オプションはセレクターとして機能し、一致するラベルを持つオブジェクトのみを取得します。**

ex)
```
$ oc get svc,dc -l app=nexus
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/nexus   ClusterIP   172.30.29.218   <none>        8081/TCP   4h

NAME                                       REVISION   DESIRED   CURRENT   ...
deploymentconfig.apps.openshift.io/nexus   1          1         1         ...
```

テンプレートを使用してリソースを生成する場合、ラベルは特に役立ちます。

テンプレートリソースには、metadata.labels セクションから分離した labels セクションがあります。labels セクションで定義されたラベルは、テンプレート自体には適用されませんが、そのテンプレートによって生成されたすべてのリソースに追加されます。 

ex)
```
apiVersion: template.openshift.io/v1
kind: Template
labels:
  app: nexus
  template: nexus-persistent-template
metadata:
...contents omitted...
  labels:
    maintainer: redhat
  name: nexus-persistent
...contents omitted...
objects:
- apiVersion: v1
  kind: Service
  metadata:
    name: nexus
    labels:
      version: 1
...contents omitted...
```

## ガイド付き演習: データベースサーバーを OpenShift へデプロイする

- openshiftクラスターにログイン
```
[student@workstation ~]$ oc login -u yasunari-tokutake -p d62b19220bc64f24970d https://api.ap45.prod.nextcle.com:6443
Login successful
...output omitted...
```

- 新規プロジェクト作成
```
[student@workstation ~]$ oc new-project yasunari-tokutake-mysql-openshift
```

- rhscl/mysql-57-rhel7 コンテナイメージから新しいアプリケーションを作成
```
[student@workstation ~]$ oc new-app --as-deployment-config --docker-image=registry.access.redhat.com/rhscl/mysql-57-rhel7 --name=mysql-openshift -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypa55 -e MYSQL_DATABASE=testdb -e MYSQL_ROOT_PASSWORD=r00tpa55
```

- 実行して新しいアプリケーションの状態を表示
```
[student@workstation ~]$ oc status
In project youruser-mysql-openshift on server https://api.cluster.lab.example.com:6443

svc/mysql-openshift - 172.30.114.39:3306
  dc/mysql-openshift deploys istag/mysql-openshift:latest
    deployment #1 running for 11 seconds - 0/1 pods
...output omitted...
```

- プロジェクト内のポッドを一覧表示
```
[student@workstation ~]$ oc get pods (-o=wide)必要であれば
```

- ポッドの詳細情報を表示
```
[student@workstation ~]$ oc describe pod mysql-openshift-1-glqrp or oc describe pod/mysql-openshift-1-glqrp
Name:               mysql-openshift-1-glqrp
Namespace:          yasunari-tokutake-mysql-openshift <- プロジェクトがNamespace
Priority:           0
PriorityClassName:  <none>
Node:               ip-10-0-148-115.ec2.internal/10.0.148.115
Start Time:         Fri, 15 Feb 2019 02:14:34 +0000
Labels:             app=mysql-openshift
                    deployment=mysql-openshift-1
                    deploymentconfig=mysql-openshift
Annotations:        openshift.io/deployment-config.latest-version: 1
                    openshift.io/deployment-config.name: mysql-openshift
                    openshift.io/deployment.name: mysql-openshift-1
                    openshift.io/generated-by: OpenShiftNewApp
                    openshift.io/scc: restricted
Status:             Running
IP:             10.129.0.85
...output omitted...
```

- ルートを作成しているサービスを公開
```
[student@workstation ~]$ oc expose service mysql-openshift
route.route.openshift.io/mysql-openshift exposed
```

- workstation とポート 3306 を使用して OpenShift 上で実行されるデータベースポッドの間にポートフォワーディングを設定します。コマンド実行後、ターミナルはハング(実行状態のまま)します。 
```
[student@workstation ~]$ oc port-forward mysql-openshift-1-glqrp 3306:3306
Forwarding from 127.0.0.1:3306 -> 3306
Forwarding from [::1]:3306 -> 3306

```


## ルートの作成

#### ルートの操作

**サービス** が OpenShift インスタンス内のポッド間でネットワークアクセスできるようにするのに対し、**ルート** には、OpenShift インスタンス外のユーザーやアプリケーションからポッドにネットワークアクセスできるようにする働きがあります。 

ルートは、外向きの IP アドレスや DNS ホスト名を、内向きのサービス IP に接続します。サービスリソースを使用してエンドポイント、つまり、サービスによって公開されるポートを見つけます。 

OpenShift ルートはクラスター全体のルーターサービスによって実装されます。これは、OpenShift クラスターでコンテナー化されたアプリケーションとして実行されます。OpenShift は、他の OpenShift アプリケーションと同じようにルーターポッドを拡張および複製します。 

注：実際には、性能を向上させて遅延を減らすため、OpenShift ルーターが直接、内部ポッドのソフトウェア定義ネットワーク (SDN) を使ってポッドに接続します。 

ルーターサービスは、デフォルトの実装として HAProxy を使用します。 

考慮するべき重要事項として、ルートに設定するパブリック DNS ホスト名は、ルーターを動かすノードの外向き IP アドレスを指す必要があります。ルーターポッドは、通常のアプリケーションポッドと違い、内部ポッド SDN ではなく、ノードのパブリック IP アドレスにバインドします。 

ex) JSON 構文で定義した最小構成のルート
```
{
    "apiVersion": "v1",
    "kind": "Route",
    "metadata": {
        "name": "quoteapp"
    },
    "spec": {
        "host": "quoteapp.apps.example.com", (ルートに結びつけられた FQDN の文字列)
        "to": {  (ルートが指すリソースを表すオブジェクトです。この場合、ルートは quoteapp に設定された name を持つ OpenShift サービスを指す。)
            "kind": "Service",
            "name": "quoteapp"
        }
    }
}
```

- MEMO(FQDN=完全修飾ドメイン名について)

![image6](images/image6.png)

#### ルートの作成

1. oc create コマンドを使用して、他の OpenShift リソースと同じように、ルートリソースを作成します。

ルートを定義する JSON または YAML リソース定義ファイルを oc create コマンドに提供する必要があります。 

oc create -f <filename>

2. oc expose service コマンドを使用して、サービスのリソース名を入力として渡し、ルートを作成することも可能です。

ex)
```
$ oc expose service quotedb --name quote
```

- デフォルトのルーティングサービスの利用

ex) ルートの確認
```
$ oc get pod --all-namespaces -l app=router
NAMESPACE          NAME                            READY  STATUS   RESTARTS  AGE
openshift-ingress  router-default-746b5cfb65-f6sdm 1/1    Running  1         4d
```

ex) oc describe pod コマンドを使用して、ルーティング設定の詳細を取得
```
$ oc describe pod router-default-746b5cfb65-f6sdm
Name:               router-default-746b5cfb65-f6sdm
Namespace:          openshift-ingress
...output omitted...
Containers:
  router:
...output omitted...
    Environment:
      STATS_PORT:                 1936
      ROUTER_SERVICE_NAMESPACE:   openshift-ingress
      DEFAULT_CERTIFICATE_DIR:    /etc/pki/tls/private
      ROUTER_SERVICE_NAME:        default
      ROUTER_CANONICAL_HOSTNAME:  apps.cluster.lab.example.com
...output omitted...
```

- MEMO(JSONについて)
```
JSONファイルは、xxx : yyy で表す

可読性が高い。(https://thinkit.co.jp/article/70/1)
```


## ガイド付き演習: サービスをルートとして公開する

- http://github.com/Yasunari0118/DO180-apps/ で Git リポジトリの php-helloworld ディレクトリから Source-to-Image を使用して、新しい PHP アプリケーションを作成
```
[student@workstation ~]$ oc new-app --as-deployment-config php:7.3~https://github.com/${RHT_OCP4_GITHUB_USER}/DO180-apps --context-dir php-helloworld --name php-helloworld

--> Found image fbe3911 (13 days old) in image stream "openshift/php" under tag "7.3" for "php:7.3"
...output omitted...
--> Creating resources ...
...output omitted...
--> Success
    Build scheduled, use 'oc logs -f bc/php-helloworld' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/php-helloworld'
    Run 'oc status' to view your app.
```

- oc get pod -w でリアルタイム情報の表示
```
[student@workstation ~]$ oc get pods -w
NAME                     READY   STATUS     RESTARTS   AGE
php-helloworld-1-build   0/1     Init:0/2   0          2s
php-helloworld-1-build   0/1   Init:0/2   0     4s
php-helloworld-1-build   0/1   Init:1/2   0     5s
php-helloworld-1-build   0/1   PodInitializing   0     6s
php-helloworld-1-build   1/1   Running   0     7s
php-helloworld-1-deploy   0/1   Pending   0     0s
php-helloworld-1-deploy   0/1   Pending   0     0s
php-helloworld-1-deploy   0/1   ContainerCreating   0     0s
php-helloworld-1-build   0/1   Completed   0     5m8s
php-helloworld-1-cnphm   0/1   Pending   0     0s
php-helloworld-1-cnphm   0/1   Pending   0     1s
php-helloworld-1-deploy   1/1   Running   0     4s
php-helloworld-1-cnphm   0/1   ContainerCreating   0     1s
php-helloworld-1-cnphm   1/1   Running   0     62s
php-helloworld-1-deploy   0/1   Completed   0     65s
php-helloworld-1-deploy   0/1   Terminating   0     66s
php-helloworld-1-deploy   0/1   Terminating   0     66s
```


## Source-to-Image を使用したアプリケーションの作成

#### Source-to-Image (S2I) プロセス

Source-to-Image (S2I) とは、**アプリケーションソースコードからコンテナーイメージを簡単にビルドできる** ツールです。

このツールは、**Git リポジトリ** からアプリケーションソースコードを取得し、**希望する言語** と**フレームワーク** に基づくベースコンテナーへこのコードをインジェクトして、アセンブルされたアプリケーションを実行する新規コンテナーイメージを作成します。 

ex)
```
[student@workstation ~]$ oc new-app --as-deployment-config php:7.3~https://github.com/${RHT_OCP4_GITHUB_USER}/DO180-apps --context-dir php-helloworld --name php-helloworld
```

S2I は、下記の理由によりOpenShift Container Platform においてアプリケーションのビルドに使用される主要な戦略です。

1. ユーザー効率: 開発者は、Dockerfile や yum install のようなオペレーティングシステムコマンドを理解する必要がありません。自分の標準のプログラミング言語ツールを使って作業を行います。

2. パッチ対応: セキュリティの問題でベースイメージにパッチが必要な場合にも、S2I は常にすべてのアプリケーションの再ビルドが可能です。たとえば、PHP ベースイメージにセキュリティの問題が見つかった場合、このイメージをセキュリティパッチで更新すると、このイメージを使用するすべてのアプリケーションがベースとして更新されます。

3. スピード: S2I を使用する場合、アセンブルプロセスで多数の複雑な操作を実行でき、ステップごとに新規レイヤを作成する必要がないため、ビルドが高速化されます。

4. エコシステム: S2I は、タイプの異なるアプリケーション間でベースイメージとスクリプトをカスタマイズしたり再利用したりできる、イメージの共有エコシステムを促進します。 


#### イメージストリームの説明

OpenShift を使用すると、新しいバージョンのユーザーアプリケーションをポッドに迅速にデプロイできます。

アプリケーションを新規作成するには、アプリケーションのソースコードだけでなく、ベースイメージ (S2I ビルダイメージ) が必要です。**これら 2 つのコンポーネントのどちらかが更新されると、OpenShift が新しいコンテナーイメージを作成します。古いコンテナイメージを使用して作成されたポッドは、新しいイメージを使用するポッドによって置き換えられます。**

アプリケーションコードが変更されたとき、コンテナーイメージを更新する必要があるのは明らかでも、ビルダーイメージが変更された場合に、デプロイされたポッドも更新する必要があるかについては明らかでないこともあります。

**イメージストリーム(is)リソースは、コンテナイメージのエイリアスであるイメージストリームタグに関連付けられた、特定のコンテナイメージを命名する設定です。** OpenShift はイメージストリームに対してアプリケーションをビルドします。OpenShift インストーラは、インストール時にデフォルトでイメージストリームをいくつか生成します。使用可能なイメージストリームを確認するには、次のように oc get コマンドを使用します。 

ex)
```
$ oc get is -n openshift
NAME           IMAGE REPOSITORY                      TAGS
cli            ...svc:5000/openshift/cli             latest
dotnet         ...svc:5000/openshift/dotnet          2.0,2.1,latest
dotnet-runtime ...svc:5000/openshift/dotnet-runtime  2.0,2.1,latest
httpd          ...svc:5000/openshift/httpd           2.4,latest
jenkins        ...svc:5000/openshift/jenkins         1,2
mariadb        ...svc:5000/openshift/mariadb         10.1,10.2,latest
mongodb        ...svc:5000/openshift/mongodb         2.4,2.6,3.2,3.4,3.6,latest
mysql          ...svc:5000/openshift/mysql           5.5,5.6,5.7,latest
nginx          ...svc:5000/openshift/nginx           1.10,1.12,1.8,latest
nodejs         ...svc:5000/openshift/nodejs          0.10,10,11,4,6,8,latest
perl           ...svc:5000/openshift/perl            5.16,5.20,5.24,5.26,latest
php            ...svc:5000/openshift/php             5.5,5.6,7.0,7.1,latest
postgresql     ...svc:5000/openshift/postgresql      10,9.2,9.4,9.5,9.6,latest
python         ...svc:5000/openshift/python          2.7,3.3,3.4,3.5,3.6,latest
redis          ...svc:5000/openshift/redis           3.2,latest
ruby           ...svc:5000/openshift/ruby            2.0,2.2,2.3,2.4,2.5,latest
wildfly        ...svc:5000/openshift/wildfly         10.0,10.1,11.0,12.0,...
```

OpenShift は、イメージストリームの変更を検出し、その変更に基づいて措置を講じます。nodejs-010-rhel7 イメージでセキュリティーの問題が発生した場合には、イメージリポジトリ内で更新できます。また、OpenShift を使用すると、自動的にアプリケーションコードの新規ビルドをトリガーできます。

組織はサポートされた Red Hat のベース S2I イメージを複数選択する可能性がありますが、独自のベースのイメージを作成することもあります。 


#### S21 と CLI によるアプリケーションのビルド

ex)
```
[student@workstation ~]$ oc new-app --as-deployment-config php:7.3~https://github.com/${RHT_OCP4_GITHUB_USER}/DO180-apps --context-dir php-helloworld --name php-helloworld

--as-deployment-config：Deployment の代わりに DeploymentConfig を作成
チルダ (~) の左側：このプロセスで使用されるイメージストリーム
チルダ (~) の右側：ソースコードの Git リポジトリの場所、ソースリポジトリ
--name：アプリケーション名を設定

もしくは、以下のように-iオプションを使用してisを指定してもいい。

$ oc new-app --as-deployment-config -i php http://services.lab.example.com/app --name=myapp
```

oc new-app コマンドを使用すると、ローカルまたはリモートの Git リポジトリからソースコードを使用してアプリケーションを作成できます。

**ソースリポジトリのみが指定されている場合、oc new-app によって、アプリケーションのビルドに使用される適切なイメージストリームの特定が試行されます。** 

アプリケーションコードに加え、S2I は、新規イメージを作成するために Dockerfile の特定と処理も行います。 

ex) 現在のディレクトリーで Git リポジトリを使用してアプリケーションを作成
```
$ oc new-app --as-deployment-config .
```

ex) リモート Git リポジトリとコンテキストサブディレクトリを使用してアプリケーション作成
```
$ oc new-app --as-deployment-config https://github.com/openshift/sti-ruby.git --context-dir=2.0/test/puma-test-app
```

ex) 特定の分岐先情報を持つリモート Git リポジトリを使用してアプリケーション作成
```
$ oc new-app --as-deployment-config https://github.com/openshift/ruby-hello-world.git#beta4
```

イメージストリームがコマンド内で指定されていない場合は、リポジトリの root 内に特定のファイルが存在するかどうかに基づき、new-app によって、どの言語ビルダを使用するかの決定が試行されます。

言語 	ファイル
Ruby 	Rakefile Gemfile、config.ru
Java EE 	pom.xml
Node.js 	app.json package.json
PHP 	index.php composer.json
Python 	requirements.txt config.py
Perl 	index.pl cpanfile

言語が検出されると、検出された言語をサポートしているイメージストリームタグ、または検出された言語の名前に一致するイメージストリームが new-app コマンドによって検索されます。 

- -o json パラメーターと出力リダイレクトを使用して JSON リソース定義ファイルを作成

ex)
```
$ oc -o json new-app --as-deployment-config php~http://services.lab.example.com/app --name=myapp > s2i.json
```

この JSON 定義ファイルがリソースの一覧を作成します。**1 番目のリソースはイメージストリームです。**

ex)
```
...output omitted...
{
    "kind": "ImageStream", 
    "apiVersion": "image.openshift.io/v1",
    "metadata": {
        "name": "myapp", (イメージストリームに myapp という名前を付けます。)
        "creationTimestamp": null
        "labels": {
                    "app": "myapp"
                },
                "annotations": {
                    "openshift.io/generated-by": "OpenShiftNewApp"
                }
    },
    "spec": {
        "lookupPolicy": {
            "local": false
        }
    },
    "status": {
        "dockerImageRepository": ""
    }
},
...output omitted...
```

BuildConfig (BC) は 2 番目のリソースで、以下の例では、OpenShift が実行可能なイメージを作成するときに使用するパラメーターの概要を示す。
BCはソースコードを実行可能なイメージに変換するときに実行される、入力パラメーターとトリガーの定義を担います。

```
...output omitted...
{
    "kind": "BuildConfig", 
    "apiVersion": "build.openshift.io/v1",
    "metadata": {
        "name": "myapp", (BCをmyappと命名)
        "creationTimestamp": null,
        "labels": {
            "app": "myapp"
        },
        "annotations": {
            "openshift.io/generated-by": "OpenShiftNewApp"
        }
    },
    "spec": {
        "triggers": [
            {
                "type": "GitHub",
                "github": {
                    "secret": "S5_4BZpPabM6KrIuPBvI"
                }
            },
            {
                "type": "Generic",
                "generic": {
                    "secret": "3q8K8JNDoRzhjoz1KgMz"
                }
            },
            {
                "type": "ConfigChange"
            },
            {
                "type": "ImageChange",
                "imageChange": {}
            }
        ],
        "source": {
            "type": "Git",
            "git": {
                "uri": "http://services.lab.example.com/app" (ソースコードのGITリポジトリのアドレス)
            }
        },
        "strategy": {
            "type": "Source", (S2I を使用する戦略を定義)
            "sourceStrategy": {
                "from": {
                    "kind": "ImageStreamTag",
                    "namespace": "openshift",
                    "name": "php:7.3" (php:7.3 イメージストリームとしてビルダーイメージを定義)
                }
            }
        },
        "output": {
            "to": {
                "kind": "ImageStreamTag",
                "name": "myapp:latest" (出力イメージストリームを myapp:latest と命名)
            }
        },
        "resources": {},
        "postCommit": {},
        "nodeSelector": null
    },
    "status": {
        "lastVersion": 0
    }
},
...output omitted...
```

3 番目のリソースは、OpenShift へのデプロイプロセスのカスタマイズを担う、デプロイ設定です。これには、新規コンテナを作成するときに必要となり、Kubernetes のレプリケーションコントローラーへ変換される、パラメーターとトリガーが含まれる場合があります。DeploymentConfigs オブジェクトが提供する機能の一例を以下に挙げます。 

1. 既存のデプロイから新規デプロイへの移行に関するユーザーカスタマイズが可能な戦略。
2. 前回のデプロイへのロールバック。
3. 手動のレプリケーションスケーリング。 

```
...output omitted...
{
    "kind": "DeploymentConfig", 
    "apiVersion": "apps.openshift.io/v1",
    "metadata": {
        "name": "myapp", 
        "creationTimestamp": null,
        "labels": {
            "app": "myapp"
        },
        "annotations": {
            "openshift.io/generated-by": "OpenShiftNewApp"
        }
    },
    "spec": {
        "strategy": {
            "resources": {}
        },
        "triggers": [
            {
                "type": "ConfigChange" (設定変更トリガにより、レプリケーションコントローラテンプレートが変更されるときにはいつでも、新規デプロイが作成されます。)
            },
            {
                "type": "ImageChange", (イメージ変更トリガーにより、新バージョンの myapp:latest イメージがリポジトリで使用できるときには常に新規デプロイが作成されます。 )
                "imageChangeParams": {
                    "automatic": true,
                    "containerNames": [
                        "myapp"
                    ],
                    "from": {
                        "kind": "ImageStreamTag",
                        "name": "myapp:latest"
                    }
                }
            }
        ],
        "replicas": 1,
        "test": false,
        "selector": {
            "app": "myapp",
            "deploymentconfig": "myapp"
        },
        "template": {
            "metadata": {
                "creationTimestamp": null,
                "labels": {
                    "app": "myapp",
                    "deploymentconfig": "myapp"
                },
                "annotations": {
                    "openshift.io/generated-by": "OpenShiftNewApp"
                }
            },
            "spec": {
                "containers": [
                    {
                        "name": "myapp",
                        "image": "myapp:latest", (デプロイするコンテナーイメージ myapp:latest を定義します。)
                        "ports": [               (コンテナのポートを指定します。 )
                            {
                                "containerPort": 8080,
                                "protocol": "TCP"
                            },
                            {
                                "containerPort": 8443,
                                "protocol": "TCP"
                            }
                        ],
                        "resources": {}
                    }
                ]
            }
        }
    },
    "status": {
        "latestVersion": 0,
        "observedGeneration": 0,
        "replicas": 0,
        "updatedReplicas": 0,
        "availableReplicas": 0,
        "unavailableReplicas": 0
    }
},
...output omitted...
```

最後は、サービスについてです。ここでは省略


- アプリケーションを新規作成した後

oc get builds コマンドを使用すると、アプリケーションビルドの一覧を表示できます。 

ex)
```
$ oc get builds
NAME               TYPE     FROM          STATUS    STARTED          DURATION
php-helloworld-1   Source   Git@9e17db8   Running   13 seconds ago
```

ビルドログも表示できる。
```
$ oc logs build/myapp-1
```

新規ビルドを oc start-build build_config_name コマンドでトリガーします。

ex)
```
$ oc get buildconfig
NAME           TYPE      FROM      LATEST
myapp          Source    Git       1

$ oc start-build myapp
build "myapp-2" started
```

#### ビルド設定とデプロイメント設定の関係

- BuildConfig ポッドの役割

OpenShift でイメージを作成し、それを内部コンテナーレジストリーへプッシュすることです。ソースコードまたはコンテンツの更新には、通常、イメージの更新を保証する新規のビルドが必要です。

- DeploymentConfig ポッドの役割

ポッドを OpenShift へデプロイすることです。

DeploymentConfig ポッドを実行した結果、内部コンテナーレジストリーにデプロイされたイメージを含むポッドが作成されます。既存の実行中のポッドは、DeploymentConfig リソースの設定方法に応じて破棄される可能性があります。

BuildConfig と DeploymentConfig のリソースは、直接やり取りすることはありません。**BuildConfig リソースはコンテナーイメージを作成または更新します。DeploymentConfig はこの新規イメージまたは更新されたイメージイベントに反応し、コンテナーイメージからポッドを作成します。**


## ガイド付き演習: Source-to-Image を使用したコンテナー化されたアプリケーションの作成

