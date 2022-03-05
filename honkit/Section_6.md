# 6. AWS CodeBuildでテスト＆ビルドする

## AWS CodeBuildとは？
 AWSで提供されているクラウドサーバー上で、プロジェクトのテストとビルドができるサービス。
 DockerやRuby、Go, pythonなどに対応。
 また、他のサービス（AWS CodeCommitやCodeDeploy）と組み合わせることも可能。
 
## AWS CodeBuildの仕組み

 AWS CodeBuildは、「ビルドプロジェクト」を元にビルドを行う。
 
 ビルドプロジェクト・・・ソースコードの指定やビルド環境の指定などビルドに関わる情報のこと。AWSコンソールで作成する情報。
 
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
  install:   ・・・インストールの段階で実行するコマンドを設定する。ビルド環境で使用するパッケージをインストールする場合に使用。
    runtime-versions:
      docker: 19   ・・・ 今回はdockerを使用しているのでdockerのバージョン19を指定。
    commands: 
  pre_build:   ・・・・ビルドの前に実行するコマンドを設定する。npmパッケージのインストールやgemのインストールする場合に使用。
    commands:
      - echo PRE_BUILD Start
      - docker-compose -f docker_compose_test.yml build   ・・・　dockerをビルドする。
  build:   ・・・ビルド時に実行するコマンドを設定する。テストを行う。
    commands:
      - echo BUILD start
      - docker-compose -f docker_compose_test.yml up -d
      - docker-compose -f docker_compose_test.yml run app bundle exec rake spec   ・・・テストを実行する。
 ```

## 今回使用するAWSのアーキテクトの確認
 
 
 
 
 
 
 
 
