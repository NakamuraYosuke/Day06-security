# Day06-security
## シークレットを適用した機密情報の管理
### シークレット
このシークレットオブジェクトタイプは、パスワード、OpenShift Container Platform クライアント設定ファイル、Docker 設定ファイル、プライベートソースリポジトリー資格情報など、機密情報を保有するメカニズ ムを提供します。

シークレットは、機密コンテンツをポッドから分離します。

ボリュームプラグインを使用してコンテナーにシークレットをマウントできます。

システムがシークレットを使用してポッドの代理でアクションを 実行することもできます。

シークレットの特徴 シークレットには主に次の特徴があります。

- シークレットデータは定義とは無関係に参照できます。
- シークレットデータボリュームは一時ファイルストレージに支えられています。
- シークレットデータはネームスペース内で共有できます。

シークレットの作成 シークレットは、シークレットに依存するポッドの前に作成します。

シークレットの作成では、次の処理を行います。
- シークレットデータを含むシークレットオブジェクトを作成します。
```
（例）
$ kubectl create secret generic secret_name \
--from-literal=key1=secret1 \
--from-literal=key2=secret2
```

#### シークレットをポッドに公開する方法
シークレットは、データボリュームとしてマウント、または環境変数として公開し、ポッドのコンテナーから使用できます。

たとえば、シークレットをポッドに公開するには、まずシークレットを作成し、username や password などの値をキー/値ペアに割り当ててから、そのキーの名前をポッドのYAMLファイルの env 定義に割り当てます。

demo-secret という名前のシークレットを作成します。

キー username を定義し、キーの値をユーザー demo-user に設定します。

```
（例）
$ kubectl create secret generic demo-secret \
--from-literal=username=demo-user
```

#### シークレットの使用例
- パスワードとユーザー名

    パスワードやユーザー名などの機密情報は、コンテナーのデータボリュームとしてマウントされるシークレッ ト内に保存できます。コンテナーのデータボリュームディレクトリーに配置されているファイル内のコンテンツ としてデータが表示されます。
    データベースなど、アプリケーションは、これらのシークレットを使用してユーザーを認証できます。

- トランスポートレイヤーセキュリティ (TLS) とキーペア

    クラスターでプロジェクトのネームスペース内のシークレットに署名済み証明書とキーのペアを生成するこ とにより、サービスへの通信のセキュリティ保護を達成できます。証明書とキーのペアは、ポッドのシークレットのデータボリュームに配置されている、tls.crt や tls.key などのファイル内で、PEM 形式を使用して保存されます。

### ConfigMapオブジェクト
ConfigMapはシークレットに似ていますが、機密情報が含まれない文字列で動作する機能をサポートするように設計されています。

ConfigMapオブジェクトは、ポッド内で消費できる、またはコントローラーなど のシステムコンポーネントの設定データの保存に使用できる設定データのキー/値ペアを保有します。

ConfigMapオブジェクトは、設定データをコンテナーにインジェクトするメカニズムを提供します。

ConfigMapは、個々のプロパティなどの緻密な情報や、設定ファイル全体や JSON BLOB などの詳細情報を保存します

#### CLI を使用した ConfigMap の作成
ConfigMapオブジェクトは、--from-literal オプションを使用してリテラル値から作成できます。

次のコマンドは、IP アドレス 172.20.30.40 を serverAddress という名前の ConfigMap キーに割り当てる ConfigMap オブジェクトを作成します。

```
（例）
$ kubectl create configmap special-config \ 
--from-literal=serverAddress=172.20.30.40
```

次のコマンドを使用して、ConfigMap を表示します。
```
$ kubectl get configmaps special-config -o yaml
```

```
apiVersion: v1
data:
  key1: serverAddress=172.20.30.40
kind: ConfigMap
metadata:
  creationTimestamp: 2017-07-10T17:13:31Z
  name: special-config
  namespace: secure-review
  resourceVersion: "93841"
  selfLink: /api/v1/namespaces/secure-review/configmapsspecial-config
  uid: 19418d5f-6593-11e7-a221-52540000fa0a
```

