# 第4章 OpenShift リソースへのアクセスの制御

ロールベースのアクセス制御を定義して適用し、機密情報をシークレットで保護する。

## RBAC を使用したパーミッションの定義と適用

#### ロールベースのアクセス制御 (RBAC)

ロールベースのアクセス制御 (RBAC) とは、コンピュータシステム内のリソースへのアクセスを管理するための手法です。

Red Hat OpenShift では、クラスターまたはプロジェクト内で特定のアクションをユーザーが実行できるかどうかは RBAC によって決定されます。ユーザーの責任レベルに応じてクラスターとローカルという 2 種類のロールを使用できます。 


#### 認可プロセス

認可プロセスは、ルール、ロール、およびバインディングによって管理されます。

- ルール : オブジェクトまたはオブジェクトグループに対して許可されるアクション。
- ロール : ルールのセット。ユーザーとグループを複数のロールに関連付けることが可能。
- バインディング : ロールに対するユーザーまたはグループの割り当て。


#### RBACのスコープ

Red Hat OpenShift Container Platform (RHOCP) では、ユーザーのスコープと責任に応じて、クラスターロールとローカルロールという、2 種類のロールおよびバインディングのグループが定義されています。

- クラスターロール : このロールレベルのユーザーまたはグループは、OpenShift クラスターを管理できます。
- ローカルロール : このロールレベルのユーザーまたはグループは、プロジェクトレベルの要素のみを管理できます。 


#### CLI を使用した RBAC の管理

クラスターロールをユーザーに追加するには、add-cluster-role-to-user サブコマンドを使用します。

> oc adm policy add-cluster-role-to-user cluster-role username

一般ユーザーをクラスター管理者に変更するには、次のコマンドを使用します。

> oc adm policy add-cluster-role-to-user cluster-admin username

クラスターロールをユーザーから削除するには、remove-cluster-role-from-user サブコマンドを使用します。

> oc adm policy remove-cluster-role-from-user cluster-role username

クラスター管理者を一般ユーザーに変更するには、次のコマンドを使用します。

> oc adm policy remove-cluster-role-from-user cluster-admin username

oc adm policy who-can コマンドを使用して、ユーザーがリソースにアクションを実行できるかどうか決めることができます。

> oc adm policy who-can delete user


#### デフォルトのロール

OpenShift には、ローカルまたはクラスター全体に割り当て可能なデフォルトのクラスターロールのセットが付属しています。
これらのロールを変更することで OpenShift リソースに対するきめ細かいアクセス制御が可能になります。

- **admin** : このロールのユーザーは、プロジェクトへのアクセス権を他のユーザーに付与するなど、すべてのプロジェクトリソースを管理できます。

- **basic-user** : このロールのユーザーには、プロジェクトの読み取りアクセス権があります。

- **cluster-admin** : このロールのユーザーには、クラスターリソースへのスーパーユーザーアクセス権があります。これらのユーザーは、クラスター上であらゆるアクションを実行でき、すべてのプロジェクトの完全制御権を持っています。

- **cluster-status** : このロールのユーザーは、クラスターのステータス情報を取得できます。

- **edit(書き込み権限)** : このロールのユーザーは、サービスやデプロイ設定など、プロジェクトから共通のアプリケーションリソースを作成、変更、削除できます。これらのユーザーが制限範囲やクォータなどの管理リソースを操作したりプロジェクトへのアクセス権を管理したりすることはできません。

- **self-provisioner** : このロールのユーザーは、新しいプロジェクトを作成できます。これはクラスターロールであり、プロジェクトロールではありません。

- **view(読み取り権限)** : このロールのユーザーは、プロジェクトリソースを表示できますが、プロジェクトリソースを変更することはできません。 


add-role-to-user サブコマンドを使用して、特定のロールをユーザーに追加します。

> oc adm policy add-role-to-user role-name username -n project

ex) wordpress プロジェクトのロール basic-user にユーザー dev を追加する。

> oc adm policy add-role-to-user basic-user dev -n wordpress


#### ユーザータイプ

- 一般ユーザー

インタラクティブなほとんどの OpenShift Container Platform ユーザーはこのユーザーで表されます。
一般ユーザーは、User オブジェクトで表されます。このタイプのユーザーは、プラットフォームへのアクセスを許可されている人を表しています。
一般ユーザーの例には、user1 や user2 があります。 

- システムユーザー

このユーザーの多くはインフラストラクチャの定義時に自動的に作成され、その主な目的はインフラストラクチャが API と安全にやり取りできるようにすることです。
システムユーザーには、クラスター管理者 (すべてにアクセス可能)、ノードごとのユーザー、ルーターとレジストリが使用するユーザーなどが含まれます。また、匿名システムユーザーが存在し、認証されていないリクエストにデフォルトで使用されます。
システムユーザーの例には、system:admin、system:openshift-registry、system:node:node1.example.com があります。 

- サービスアカウント

プロジェクトに関連付けられる特別なシステムユーザー。一部は、プロジェクトが最初に作成されるときに自動的に作成され、プロジェクト管理者が、各プロジェクトのコンテンツへのアクセスを定義する目的でも作成できます。
サービスアカウントは、多くの場合、ポッドまたはデプロイ設定に追加の権限を付与するために使用されます。サービスアカウントは、ServiceAccount オブジェクトで表されます。
サービスアカウントユーザーの例には、system:serviceaccount:default:deployer や system:serviceaccount:foo:builder があります。 


#### 演習

self-provisioner クラスターロールを参照するすべてのクラスターロールバインディングを一覧表示します。 

> oc get clusterrolebinding -o wide | grep -E 'NAME|self-provisioner'

ex) self-provisioner クラスターロールが system:authenticated:oauth グループに割り当てられていることを確認します。 
```
[student@workstation ~]$ oc describe clusterrolebinding(s) self-provisioners
Name:         self-provisioners
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  self-provisioner
Subjects:
  Kind   Name                        Namespace
  ----   ----                        ---------
  Group  system:authenticated:oauth
```

self-provisioner クラスターロールを system:authenticated:oauth 仮想グループから削除します。

> oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth

authorization-rbac プロジェクトに関して、leader ユーザーにプロジェクト管理権限を付与します。 

> oc policy add-role-to-user admin leader

dev-group という名前のグループを作成します。 

> oc adm groups new dev-group

developer ユーザーを dev-group に追加します。

> oc adm groups add-users dev-group developer



## シークレットを使用した機密情報の管理

- 機密情報を管理するためのシークレットを作成して適用する。
- アプリケーション間でシークレットを共有する。 

#### シークレットの概要

最新のアプリケーションは、コード、設定、およびデータが疎結合の設計になっています。設定ファイルとデータがソフトウェアの一部としてハードコーディングされているわけではありません。代わりに、ソフトウェアは外部ソースから設定とデータをロードします。これにより、アプリケーションソースコードを変更せずに、異なる環境にアプリケーションをデプロイできるようになります。 

アプリケーションの多くが、機密情報へのアクセスを必要としています。たとえば、バックエンドの Web アプリケーションでは、データベースのクエリーを実行するために、データベースの資格情報へのアクセスが必要になります。

Kubernetes と OpenShift は、シークレットリソースを使用して、次のような機密情報を保持します。

- パスワード
- 機密性の高い設定ファイル
- SSH キーや OAuth トークンなど、外部リソースへの資格情報

**シークレットには、どのような種類のデータも保存できます。**
シークレット内のデータは Base64 でエンコードされているため、プレーンテキストでは保存されません。シークレットデータは暗号化されません。シークレットを Base64 形式からデコードすると元のデータにアクセスできます。 


#### シークレットの特徴

- シークレットデータはプロジェクト名前空間内で共有できます。

- シークレットデータはシークレット定義とは無関係に参照されます。管理者は、シークレットリソースを作成および管理できます。他のチームメンバーは、それぞれのデプロイ設定内でシークレットを参照します。

- OpenShift がポッドを作成すると、シークレットデータがポッドに挿入されます。シークレットは、環境変数として、またはポッド内のマウント済みファイルとして公開できす。

- ポッドの実行中にシークレットの値が変更されても、そのシークレットデータがポッド内で更新されることはありません。シークレット値が変更された後は、新しいポッドを作成て新しいシークレットデータを挿入する必要があります。

- OpenShift がポッドに挿入するシークレットデータは、いずれも一時的なものです。OpenShift が機密データを環境変数としてポッドに公開している場合、それらの変数は、ッドが破棄されると破棄されます。


#### シークレットの使用例

シークレットの主な使用例として、**資格情報の保存とサービス間通信の保護**という 2 つの用途があります。

- 資格情報

機密情報 (パスワードやユーザー名など) をシークレットに保存します。 

アプリケーションがファイルから機密情報を読み取ることが想定される場合には、シークレットをデータボリュームとしてポッドにマウントします。アプリケーションは、シークレットを通常のファイルとして読み取って機密情報にアクセスできます。たとえば、データベースによっては、ファイルから資格情報を読み取ってユーザーを認証する場合があります。

一部のアプリケーションは、環境変数を使用して設定と機密データを読み取ります。シークレット変数は、デプロイ設定内のポッド環境変数にリンクさせることができます。 

- トランスポートレイヤーセキュリティー (TLS) とキーペア

クラスターでプロジェクトの名前空間内のシークレットに署名済み証明書とキーのペアを生成することにより、サービスへの通信を保護できます。

証明書とキーのペアは、ポッドのシークレットのデータボリュームに配置されている tls.crt や tls.key などのファイル内で、PEM 形式を使用して保存されます。 


#### シークレットの作成

シークレットデータを含むシークレットオブジェクトを作成します。 

> oc create secret generic secret_name --from-literal key1=secret1 --from-literal key2=secret2

ポッドのサービスアカウントを更新して、シークレットへの参照を許可します。たとえば、特定のサービスアカウントの下で実行するポッドからシークレットをマウントできるようにするには、次のように指定します。 

> oc secrets add --for mount serviceaccount/serviceaccount-name secret/secret_name


#### ポッドへのシークレットの公開

シークレットをポッドに公開するには、まずシークレットを作成します。機密データの各部分をキーに割り当てます。作成後のシークレットには、キーと値のペアが含まれています。 

次のコマンドによって、2 つのキー (値が demo-user である username と、値が =zT1KTgk である root_password) を持つ demo-secret という名前の汎用シークレットが作成されます。シークレットには、キーと値のペアが 1 つ含まれます。

> oc create secret generic demo-secret --from-literal username=demo-user --from-literal root_password=zT1KTgk 


- ポッド環境変数としてのシークレット

デプロイ設定の環境変数セクションを変更して、シークレットの値が使用されるようにします。 
```
env:
  - name: MYSQL_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: demo-secret
        key: root_password
```

oc set env コマンドを使用して、シークレット値からアプリケーション環境変数を設定することもできます。 

> oc set env dc/demo --from=secret/demo-secret


- ポッド内のファイルとしてのシークレット

次のコマンドにより、シークレットのコンテンツを含むポッド内にボリュームマウントが作成されます。 

> oc set volume dc/demo --add --type=secret --secret-name=demo-secret --mount-path=/app-secrets (シークレットデータを、ポッド内の /app-secrets ディレクトリで使用できるようにします。)


#### 演習

MySQL データベースにアクセスするための資格情報と接続情報が含まれるシークレットを作成します。

> oc create secret generic mysql --from-literal user=myuser --from-literal password=redhat123 --from-literal database=test_secrets --from-literal hostname=mysql 

上記のシークレットを使用して、mysql デプロイ設定の環境変数を初期化します。
初期化を成功させるには、デプロイに MYSQL_USER、MYSQL_PASSWORD、MYSQL_DATABASE の各環境変数が必要になります。
シークレットには、user キー、password キー、database キーがあります。
これらは、環境変数としてデプロイ設定に割り当てることができ、MYSQL_ プレフィックスを追加します。
**シークレットを事前に作成し上で、Prefixで必要な接頭子？をつけてあげれば、デプロイ設定の環境変数に代入できる。**

> oc set env dc/mysql --prefix MYSQL_ --from secret/mysql
> oc set env dc/quotes --prefix QUOTES_ --from secret/mysql

- memo(外部公開から確認まで)
> oc expose service quotes (quotesというアプリケーションサービスを外部に公開)
> oc get route quotes (公開したアプリケーションサービスのホスト名を確認)

ex)
```
NAME     HOST/PORT                                                        ...
quotes   quotes-authorization-secrets.apps.cluster.domain.example.com     ...
```
> curl http://quotes-authorization-secrets.apps.cluster.domain.example.com/RESTエントリーポイント (アプリケーションにアクセス)
```
oc logs pod/xxx(RUNNING)でRESTエントリーポイント(env, status, random)を確認できる。

ex) 
starting application
Services:
/
/random
/env
/status
```


## セキュリティーコンテキスト制約 (SCC) を使用したアプリケーションパーミッションの制御

#### セキュリティーコンテキスト制約 (SCC)

**SCCは、リソースへのアクセスは制限するが OpenShift の操作は制限しないセキュリティー方式です。**

SCC は、OpenShift で実行中のポッドからホスト環境へのアクセスを制限します。SCC によって、次の操作が制御されます。

- 特権コンテナーの実行
- コンテナーに対する追加機能のリクエスト
- ホストディレクトリのボリュームとしての使用
- コンテナーの SELinux コンテキストの変更
- ユーザー ID の変更 

クラスター管理者として次のコマンドを使用することで、OpenShift が定義する SCC を一覧表示できます。

> oc get scc

OpenShift は、8 つの SCC を提供します。

- anyuid
    - run as user 戦略が RunAsAny として定義され、これは、コンテナーで利用できる任意のユーザー ID としてポッドが実行できることを意味します。これにより、特定のユーザーを必要とするコンテナーが、特定のユーザー ID を使用してコマンドを実行できます。
- hostaccess
- hostmount-anyuid
- hostnetwork
- node-exporter
- nonroot
- privileged
- restricted 
    - OpenShift が作成するすべてのコンテナーは restricted という SCC を使用します。これは、OpenShift 外部にあるリソースへのアクセスを制限します。


SCC の追加情報を取得するには、oc describe コマンドを使用します。
```
[user@demo ~]$ oc describe scc anyuid
Name:           anyuid
Priority:         10
Access:
  Users:          <none>
  Groups:         system:cluster-admins
Settings:
  Allow Privileged:       false
  Default Add Capabilities:     <none>
  Required Drop Capabilities:     MKNOD,SYS_CHROOT
  Allowed Capabilities:       <none>
  Allowed Volume Types:       configMap,downwardAPI,emptyDir,persistentVolumeClaim,secret
  Allow Host Network:       false
  Allow Host Ports:       false
  Allow Host PID:       false
  Allow Host IPC:       false
  Read Only Root Filesystem:      false
  Run As User Strategy: RunAsAny
    UID:          <none>
    UID Range Min:        <none>
    UID Range Max:        <none>
  SELinux Context Strategy: MustRunAs
    User:         <none>
    Role:         <none>
    Type:         <none>
    Level:          <none>
  FSGroup Strategy: RunAsAny
    Ranges:         <none>
  Supplemental Groups Strategy: RunAsAny
    Ranges:         <none>
```

異なる SCC を使用して実行されるようコンテナーを変更するには、ポッドにバインドされるサービスアカウントを作成する必要があります。
サービスアカウントを作成するには、次のコマンドを使用します。 

> oc create serviceaccount(sa) service-account-name

サービスアカウントを SCC と関連付けるには、次のコマンドを使用します。 

> oc adm policy add-scc-to-user SCC -z service-account-name

高度なセキュリティー要件を必要とするポッドをどのアカウントで作成できるか特定するには、 scc-subject-review サブコマンドを使用します。これにより、コンテナーの制約を克服するために使用できるすべてのSCCが返されます。次に例を示します
s
> oc get pod podname -o yaml | oc adm policy scc-subject-review -f -


#### 特権コンテナー

コンテナーによっては、ホストのランタイム環境にアクセスする必要があります。たとえば、S2I ビルダーコンテナーは、固有コンテナーの制限を超えたアクセス権を必要とする特権コンテナーの一種です。これらのコンテナーは、OpenShift ノード上でどのようなリソースでも使用できるため、セキュリティーリスクをもたらす可能性があります。SCC を使用して、特権アクセスを付与したサービスアカウントを作成することにより、特権コンテナーのアクセス権を有効化できます。 


#### 演習


## オープンラボ

self-provisioner クラスターロールを system:authenticated:oauth 仮想グループから削除します。

> oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth

wp-mgrs という名前のグループを作成します。

> oc adm groups new wp-mgrs

クラスター作成権限を wp-mgrs グループに付与します。

> oc adm policy add-cluster-role-to-group self-provisioner wp-mgrs

leader ユーザーを wp-mgrs グループに追加します。

> oc adm groups add-users wp-mgrs leader

authorization-review プロジェクトに関して、wp-devs グループに編集権限を付与します。 

> oc policy add-role-to-group edit wp-devs

wordpress-sa という名前のサービスアカウントを作成します。 

> oc create sa wordpress-sa

wordpress-sa サービスアカウントに anyuid SCC を付与します。

> oc adm policy add-scc-to-user anyuid -z wordpress-sa

