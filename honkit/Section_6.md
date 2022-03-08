# 6. AWS CodeBuildでテスト＆ビルドする

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
version: 0.2　   ・・・buildspecのバージョンを指定（最新の0.2を使用）

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

実際の現場では、テスト環境用のDBが用意されていることがあると思います。Codebuildは、VPC内に設置することも可能です。  
同じVPC内にRDSを設置することでCodeBuildからRDSに接続してテストを実行することができます。  
**今回は実際の現場で使えそうな利用方法として、テスト用のRDSに接続してビルドを実行したいと思います。**


### 今回使用するAWSのアーキテクトの確認

 ![スクリーンショット 2022-03-06 18 17 09](https://user-images.githubusercontent.com/52161269/156916843-075d9920-4f8b-4d75-b94e-e7cf2be1489d.png)

### ポイント
- VPCのパブリックサブネットにはNatGatewayを設置し、インターネット接続を可能にしておく
- CodeBuildとRDSはプライベートサブネットに設置する
- RDSのセキュリティグループに、CodeBuildに設定しいるセキュリティグループからのアクセスを追加する。（この設定によってCodeBuildはRDSに接続可能になる）

### ビルドプロジェクトの作成

ビルドプロジェクトでVPCを指定する必要があるので、事前にVPC・サブネット・RDSを作成しておきましょう。  
AWS Codebuildのビルドプロジェクトの作成画面です。

![スクリーンショット 2022-03-06 21 45 21](https://user-images.githubusercontent.com/52161269/156923853-9b0e6438-74fc-46dc-9cef-fe34e57a9a06.png)  
 - プロジェクト名・・・ プロジェクト名を指定する（今回はmirameetVol28)

![スクリーンショット 2022-03-06 21 47 48](https://user-images.githubusercontent.com/52161269/156923973-e0f640e9-7487-46a1-ad83-ac1ff015b7d8.png)  
- ソースプロバイダ・・・GitHubやBitbucketなど　ソース共有ツールを指定する（今回はGitHub)
 - リポジトリ・・・ソースのリポジトリの種別（PublicリポジトリかPrivateリポジトリか）を選択（今回のmirameetVol28はパブリックリポジトリ）
 - GitHubリポジトリ・・・対象のリポジトリを選択
 
 ![スクリーンショット 2022-03-06 22 03 50](https://user-images.githubusercontent.com/52161269/156924513-55b56734-d3fb-4582-81fc-ef74c83fb045.png)  
- 環境イメージ・・・AWSの用意している環境かDockerを指定する
- オペレーティングシステム・・・OSを指定する（今回は、Ubuntu)
- ランタイム・・・ランタイムを選択（Standardを選択）
- イメージ・・・イメージを選択(aws/codebuild/standard:4.0)
- 特権付与・・・docker-composeでビルドする際はtチェックを入れる


![スクリーンショット 2022-03-06 22 49 09](https://user-images.githubusercontent.com/52161269/156926221-27c45b16-ee1d-4682-8bb9-90ca5b5aa617.png)
![スクリーンショット 2022-03-06 22 47 25](https://user-images.githubusercontent.com/52161269/156926225-3d36c51c-3612-4be2-8d7f-4d506f4c143a.png)  
- VPC・・・事前に作成したVPCを選択
- サブネット・・・プライベートサブネットを選択
- セキュリティグループ・・・CodeBuild用のセキュリティグループを選択
- 環境変数・・・環境変数を設定（ビルド環境に設定される）

![スクリーンショット 2022-03-06 22 20 31](https://user-images.githubusercontent.com/52161269/156925549-8289f9b2-b85b-4f9f-bca6-1bc8ad5f5dbb.png)  
 - ビルド仕様・・・ビルドコマンドの設定（今回はBuildspecファイルを使用する）

![スクリーンショット 2022-03-07 19 34 43](https://user-images.githubusercontent.com/52161269/157014956-a14dd54d-e5f5-4b5b-880f-aa45673c590b.png)

最後に、ビルドプロジェクトを作成するをクリックして作成します！

## CodeBuildでビルドを実行する

1. ビルドプロジェクト一覧から、プロジェクをクリック
2. 「ビルドを開始」ボタンをクリック

ビルドには、５分ほどかかります。









