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

#### シークレットの分類
一口にSecretといっても、いくつかのタイプに分けられています。通常のパスワードなどはGeneric用の「type: Opeque」を利用します。

他にも、Ingressなどで参照可能なTLS用のSecretや、Docker Registryへの認証情報用のSecretなどもあります。

また、手動で作ることはありませんが、PodにService AccountのTokenをマウントするための、「type: kubernetes.io/service-account-token」のSecretも存在します。

- Generic（type: Opaque）
- TLS（type: kubernetes.io/tls）
- Docker Registry（type: kubernetes.io/dockerconfigjson）
- Service Account（type: kubernetes.io/service-account-token）


#### Genericタイプのシークレット
一般的なフリースキーマのSecretを作成する場合は、genericを指定します。作成方法は下記の4パターンがあります。

- ファイルから作成する（--from-file）
- yamlファイルから作成する
- kubectlから直接作成する（--from-literal）
- envfileから作成する

Secretでは、Secretの名前の中に、複数のKey-Value値が保存されます。

例えば、Databaseの認証情報を作成する場合、Secret名はdb-auth、Keyはusername、passwordの2種類となります。

もちろん、Kubernetesクラスタ内で複数のDBを作成する場合もあると思うので、その場合はSecret名をユニークとなるように定義するか、システムごとにNamespaceを分割する必要があります。

#### TLSタイプのシークレット
証明書として利用するSecretを作成する場合は、tlsを指定します。

TLSタイプのSecretはIngressリソースなどから利用することが一般的です。

TLSタイプの場合には、基本的にファイルから作成することが望ましいです。

```
（例）
# 証明書の作成
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /tmp/tls.key -out /tmp/tls.crt -subj "/CN=sample1.example.com"
 
# TLS Secret の作成
$ kubectl create secret tls tls-sample --key /tmp/tls.key --cert /tmp/tls.crt
```

#### Docker Registryタイプのシークレット
Docker RegistryタイプのSecretを作成する場合は、docker-registryを指定します。

kubectlから作成する際には、Registryサーバと認証情報を引数で指定します。
```
（例）
kubectl create secret docker-registry sample-registry-auth \
  --docker-server=REGISTRY_SERVER \
  --docker-username=REGISTRY_USER \
  --docker-password=REGISTRY_USER_PASSWORD \
  --docker-email=REGISTRY_USER_EMAIL
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

## 演習
`secure-secrets`という名前のnamespaceを作成し切り替えます。

```
$ kubectl config use-context crc-developer
$ kubectl create ns secure-secrets
$ kubectl config set-context crc-developer --namespace=secure-secrets
```

MySQLのアカウント情報を含むシークレットファイルを作成します。

userにmyuserを、passwordにredhat123を、databaseにtest_secretsを、hostnameにmysqlを設定します。

```
$ kubectl create secret generic mysql \
--from-literal user=myuser \
--from-literal password=redhat123 \
--from-literal database=test_secrets \
--from-literal hostname=mysql
```

MySQLを作成するdeploymentファイル（`mysql-deployment.yaml`）を作成し、applyします。
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      deployment: mysql
  template:
    metadata:
      labels:
        deployment: mysql
    spec:
      containers:
      - env:
        image: registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47
        name: mysql
```

```
$ kubectl apply -f mysql-deployment.yaml
```

Podのステータスを確認します。
```
$ kubectl get pods -w
NAME                    READY   STATUS             RESTARTS   AGE
mysql-d9cddc8f6-f9xv2   0/1     CrashLoopBackOff   1          7s
```

`CrashLoopBackOff`となり、Podの作成に失敗します。
Deploymentに MYSQL_USER、MYSQL＿PASSWORD、MYSQL_DATABASEなどの環境変数を設定していないためです。

以下の内容の`mysql-secret-deployment.yaml`を作成しapplyます。
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      deployment: mysql
  template:
    metadata:
      labels:
        deployment: mysql
    spec:
      containers:
      - env:
        - name: MYSQL_HOSTNAME
          valueFrom:
            secretKeyRef:
              key: hostname
              name: mysql
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: mysql
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              key: user
              name: mysql
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              key: database
              name: mysql
        image: registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47
        name: mysql
```

```
$ kubectl apply -f mysql-secret-deployment.yaml
```
Podのステータスを確認します。
```
$ kubectl get pods -w
NAME                     READY   STATUS              RESTARTS   AGE
mysql-6ddbc5566f-f7xzn   0/1     ContainerCreating   0          3s
mysql-6ddbc5566f-f7xzn   1/1     Running             0          4s
```

`Running`となっていることを確認します。

作成したMySQLのPodにログインします。
```
$ kubectl exec -it mysql-6ddbc5566f-f7xzn /bin/bash
```

シークレットに登録したユーザ、パスワード、データベース名でログインを試みます。

```
bash-4.2$ mysql -umyuser -predhat123 test_secrets -e 'show databases;'
```

次に`quotes`というアプリケーションを用いてMySQLのシークレット情報を表示します。

以下の内容の`quotes-deployment.yaml`を作成しapplyします。
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quotes
spec:
  selector:
    matchLabels:
      deployment: quotes
  template:
    metadata:
      labels:
        deployment: quotes
    spec:
      containers:
      - env:
        - name: QUOTES_DATABASE
          valueFrom:
            secretKeyRef:
              key: database
              name: mysql
        - name: QUOTES_HOSTNAME
          valueFrom:
            secretKeyRef:
              key: hostname
              name: mysql
        - name: QUOTES_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: mysql
        - name: QUOTES_USER
          valueFrom:
            secretKeyRef:
              key: user
              name: mysql
        image: nakanakau/famous-quotes:1.0
        name: quotes
        ports:
        - containerPort: 8000
          protocol: TCP
```

```
$ kubectl apply -f quotes-deployment.yaml
```

Podの状態を確認します。
```
$ kubectl get pod -w
NAME                      READY   STATUS    RESTARTS   AGE
mysql-6ddbc5566f-4c2ck    1/1     Running   0          20m
quotes-5b649bb4f4-khlwx   0/1     Error     0          4s
```

`quotes`のPodが`Error`となり失敗しています。ログを確認します。
```
$ kubectl logs -f quotes-5b649bb4f4-khlwx
2021/04/04 16:14:26 Connecting to the database: myuser:redhat123@tcp(mysql:3306)/test_secrets
2021/04/04 16:14:27 Database connection error: dial tcp: lookup mysql on 10.217.4.10:53: no such host
```

MySQL Databaseへの接続に失敗しています。

これはMySQLのServiceを作成していないためです。

MySQLのServiceを`mysql-service.yaml`という名前で作成しapplyします。
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: mysql
  name: mysql
  namespace: secure-secrets
spec:
  ports:
  - name: 3306-tcp
    port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    deployment: mysql
  type: NodePort
```

```
$ kubectl apply -f mysql-service.yaml
```

`quotes`のDeploymentを削除し再度applyします。
```
$ kubectl delete deployment quotes
$ kubectl apply -f quotes-deployment.yaml
```

再度Podの状態を確認します。
```
$ kubectl get pod -w
NAME                      READY   STATUS    RESTARTS   AGE
mysql-6ddbc5566f-4c2ck    1/1     Running   0          27m
quotes-5b649bb4f4-qnhlq   1/1     Running   0          2m28s
```

次にquotesのアプリケーションに対するServiceも`quotes-service.yaml`という名前で作成しapplyします。
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: quotes
  name: quotes
  namespace: secure-secrets
spec:
  ports:
  - name: 8000-tcp
    port: 8000
    protocol: TCP
    targetPort: 8000
  selector:
    deployment: quotes
  type: NodePort
```

```
$ kubectl apply -f quotes-service.yaml
```

作成したquotesのServiceを確認します。
```
$ kubectl get svc quotes
NAME     TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
quotes   NodePort   10.217.4.31   <none>        8000:30121/TCP   6s
```

quotesアプリケーションにアクセスします。
`192.168.64.2`はCRCのIPアドレスとなります。
```
$ curl http://192.168.64.2:30121/status
Database connection OK
```

```
$ curl http://192.168.64.2:30121/env
<html>
	<head>
        <title>Quotes</title>
    </head>
    <body>
       
        <h1>Quote List</h1>
        
            <h3>Database Environment Variables</h3>
            <ul>
                <li>QUOTES_USER: myuser </li>
                <li>QUOTES_PASSWORD: redhat123 </li>
                <li>QUOTES_DATABASE: test_secrets</li>
                <li>QUOTES_HOST: mysql</li>
            </ul>
        
    </body>
</html>
```

MySQLへの接続も正常に行われ、シークレットに設定した情報も確認できました。