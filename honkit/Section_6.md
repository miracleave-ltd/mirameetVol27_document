# AWS CodeBuildでテスト＆ビルドする

## 本セクションで行うこと
- AWS CodeBuildの解説
- ビルドプロジェクトを作成する
- CodeBuildでビルドを実行する

## AWS CodeBuildの解説
 AWSで提供されているクラウドサーバー上で、プロジェクトのテストとビルドができるサービス。  
 Rubyの他にJava、Go, pythonなどに対応。  
 また、他のサービス（AWS CodeCommitやCodeDeploy）と組み合わせて利用することも可能。
 
 ### AWS CodeBuildの仕組み

 AWS CodeBuildは、「ビルドプロジェクト」を元にビルドを行う。  
 ビルドプロジェクト・・・ソースコードの指定やビルド環境の指定などビルドに関わる情報のこと。AWSコンソールから作成できる。
 
 ### AWS CodeBuildの処理の流れ
 AWSコンソールからCodeBuildを実行すると、以下の流れでビルドが行われる。
   
<img width="548" alt="スクリーンショット 2022-03-05 22 06 11" src="https://user-images.githubusercontent.com/52161269/156884332-383af062-3ca2-4af9-b96f-d16baf62bfb7.png">

 1. ビルドプロジェクトに基づきビルド環境を作成
 2. ビルド環境で、ソースコードをダウンロード後、buildspec.ymlで設定したコマンドを実行してソースのコンパイル・テスト・ビルドを行う。
 3. ビルド出力がある場合は、S3にデプロイ可能なアーティファクトをアップロードされる。
 
 １〜３のログは、AWS CloudWatch Logsに送信される。
 
### buildspec.yml
  ビルド環境で実行するコマンドなどを記載するyaml形式のファイル。  

  ```
  version: 0.2   ・・・buildspecのバージョンを指定（最新の0.2を使用）
  phases:
    install:   ・・・インストールの段階で実行するコマンドを設定する。主にビルド環境で使用するパッケージのインストールなどに使用。
      runtime-versions:
        docker: 19   ・・・ 今回はdockerを使用しているのでdockerのバージョン19を指定（※１）。
      commands: 
    pre_build:   ・・・・ビルドの前に実行するコマンドを設定する。主にnpmパッケージのインストールやgemのインストールなどに使用。
        - echo PRE_BUILD Start
        - docker-compose -f docker_compose_test.yml build   ・・・　dockerをビルドする。
    build:   ・・・ビルド時に実行するコマンドを設定する。主にテストを行う。
      commands:
        - echo BUILD start
        - docker-compose -f docker_compose_test.yml up -d
        - docker-compose -f docker_compose_test.yml run app bundle exec rake spec   ・・・テストを実行する。
  ```

※１[バージョンはこちらを参照](https://github.com/aws/aws-codebuild-docker-images/blob/master/ubuntu/standard/4.0/runtimes.yml)


## ビルドプロジェクトを設定する

実際の現場では、テスト環境用のDBが用意されていることがあると思います。  
CodebuildはVPC内に設置することも可能なので、同じVPC内にRDSを設置することでCodeBuildからRDSに接続してテストを実行することができます。  

**今回は実際の現場で使えそうな利用方法として、テスト用のRDSに接続してビルドを実行したいと思います。**

***

### 今回使用するAWSのアーキテクトの確認

 ![スクリーンショット 2022-03-06 18 17 09](https://user-images.githubusercontent.com/52161269/156916843-075d9920-4f8b-4d75-b94e-e7cf2be1489d.png)

**ポイント**
- VPCのパブリックサブネットにはNatGatewayを設置し、インターネット接続を可能にしておく
- CodeBuildとRDSはプライベートサブネットに設置する
- RDSのセキュリティグループに、CodeBuildを設定するサブネットからのアクセスを追加する。（この設定によってCodeBuildはRDSに接続可能になる）

**CodeBuild以外の上記のアーキテクトは、CloudFormationを用意しているのでサクッと作成していきましょう！**
***
#### AWSにログイン & CloudFormationのスタック作成画面に移動

AWSにログイン後に、上部の検索フォームでCloudFormationのサービスへ移動します。
![スクリーンショット 2022-04-03 16 08 53](https://user-images.githubusercontent.com/52161269/161416093-915dba3c-5b1d-4751-867b-5dc56e578acc.png)

「スタックの作成」をクリックすれば、スタック作成画面へ遷移します。
![スクリーンショット 2022-04-03 16 19 08](https://user-images.githubusercontent.com/52161269/161416368-1827bb55-a70b-4960-bf11-e7c735b24159.png)

#### テンプレートの指定

最初にvpcのスタックを作成します。  
`テンプレートファイルのアップロード`を選択、project直下にある`vpc.yml`を選択し、「次へ」をクリックします。  
ファイルパス:
```
mirameetVol27/vpc.yml
```
![スクリーンショット 2022-04-03 19 02 38](https://user-images.githubusercontent.com/52161269/161422485-e469bd29-fb3a-4dbb-84c1-09eaaf34f0d2.png)


#### スタックの詳細設定

「スタックの名前」は`mirameet-27-vpc`パラメータの「PJPrefix」は`mirameet`とし、「次へ」をクリックします。
![スクリーンショット 2022-04-03 19 08 02](https://user-images.githubusercontent.com/52161269/161422439-58e54002-3e9d-4439-b3b8-0ada6d8f3060.png)


#### スタックの作成
ステップ４のレビューまで「次へ」をクリックします。ステップ４まできたら下部にある「スタックの作成」をクリックします。

#### RDSのスタック作成
スタックの作成が完了したら、RDSのスタックを作成します。  
上記と同じ流れで作成します。  
テンプレートは`rds.yml`  
ファイルパス：  
```
/mirameetVol27/rds.yml
```
「スタックの名前」は`mirameet-27-rds`で作成します。（パラメータの「PJPrefix」は指定する必要はありません。）


***

### ビルドプロジェクトの作成

ビルドプロジェクトでVPCを指定する必要があるので、事前にVPC・サブネット・RDSを作成しておきましょう。  
AWS Codebuildのビルドプロジェクトの作成画面です。

![スクリーンショット 2022-03-28 15 43 07](https://user-images.githubusercontent.com/52161269/160341003-33a70d0e-9a81-46aa-a87f-0f753e34a2c5.png)
- プロジェクト名・・・ プロジェクト名を指定する（今回はmirameetVol27)
  
![スクリーンショット 2022-03-28 15 44 49](https://user-images.githubusercontent.com/52161269/160341256-61fa7819-164b-43b3-a197-855e81a8c67d.png)
- ソースプロバイダ・・・GitHubやBitbucketなど　ソース共有ツールを指定する（今回はGitHub)
 - リポジトリ・・・ソースのリポジトリの種別（PublicリポジトリかPrivateリポジトリか）を選択  
（今回のmirameetVol27はパブリックリポジトリ）

```
https://github.com/miracleave-ltd/mirameetVol27
```

 - GitHubリポジトリ・・・対象のリポジトリを選択
 
 ![スクリーンショット 2022-03-06 22 03 50](https://user-images.githubusercontent.com/52161269/156924513-55b56734-d3fb-4582-81fc-ef74c83fb045.png)  
- 環境イメージ・・・AWSの用意している環境かDockerを指定する
- オペレーティングシステム・・・OSを指定する（今回は、Ubuntu)
- ランタイム・・・ランタイムを選択（Standardを選択）
- イメージ・・・イメージを選択(aws/codebuild/standard:4.0)
- 特権付与・・・docker-composeでビルドする際はチェックを入れる


![スクリーンショット 2022-03-06 22 49 09](https://user-images.githubusercontent.com/52161269/156926221-27c45b16-ee1d-4682-8bb9-90ca5b5aa617.png)
![スクリーンショット 2022-03-28 15 22 18](https://user-images.githubusercontent.com/52161269/160338195-a1f79fab-597a-47d3-bab3-61a9b99094ce.png)
- VPC・・・事前に作成したVPCを選択
- サブネット・・・プライベートサブネットを選択
- セキュリティグループ・・・CodeBuild用のセキュリティグループを選択
- 環境変数・・・環境変数を設定（ビルド環境に設定される）

![スクリーンショット 2022-03-06 22 20 31](https://user-images.githubusercontent.com/52161269/156925549-8289f9b2-b85b-4f9f-bca6-1bc8ad5f5dbb.png)  
 - ビルド仕様・・・ビルドコマンドの設定（今回はBuildspecファイルを使用する）

![スクリーンショット 2022-03-07 19 34 43](https://user-images.githubusercontent.com/52161269/157014956-a14dd54d-e5f5-4b5b-880f-aa45673c590b.png)

最後に、ビルドプロジェクトを作成するをクリックして作成します！

## CodeBuildでビルドを実行する

1. ビルドプロジェクト一覧から、プロジェクトをクリック
2. 「ビルドを開始」ボタンをクリック

ビルドには、５分ほどかかります。









