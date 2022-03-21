# 結合テスト（IT）
## 本セクションで行うこと
1. 今回取り上げる結合テストについて
2. Request specの解説
3. 本アプリでの実例紹介（後で変更するかも）


## 今回取り上げる結合テストについて

結合テストは、Integration Test(統合テスト)とも呼ばれ、システム開発において単体レベルで開発した機能が組み合わさって動作した時、期待される動作をするかどうかをテストします。  
結合テストは、実施する対象によって内部結合・外部結合に分かれます。  

**内部結合**・・・ システム内部の機能同士を組み合わせてテストを行う  
**外部結合**・・・ システム内部の機能とシステム外部の機能を組み合わせてテストを行う

今回取り上げる結合テストは「内部結合」になります
<img width="705" alt="スクリーンショット 2022-03-13 21 01 04" src="https://user-images.githubusercontent.com/52161269/158058448-6788ff57-03d4-4de7-b51e-cdd24fe626f3.png">


今回の投稿アプリでは、以下３つの機能があります。
1. ユーザー機能(users_controller)  
  ユーザー作成、ユーザー認証（ログイン・ログアウト）
2. 投稿機能(posts_controller)  
  投稿の新規作成、投稿の編集、投稿の一覧表示、投稿削除
3. コメント機能(comments_controller)  
  対象の投稿に対してコメントの作成

内部結合では、３つの機能の間の連携部分が期待される動きをするか？というテストをします。
具体的な観点は以下です。
- コントローラーへのリクエストが通ること
- 期待されるレスポンスが返却されること


## Request specの解説

Request specを使うと、以下のテストができるようになります。

- HTTPリクエストのテストができる
- 複数コントローラーの複数のリクエストがテストできる
- 複数のセッションでリクエストを指定できる


以下、Request specファイルの生成コマンド
```
rails g rspec:request ファイル名
```

今回は投稿機能の結合テスト(posts_spec.rb)を例に実際のテストコードを確認していきます。  

これから説明するテストの内容は、以下です。  
`ユーザーがログインしている場合は投稿一覧のリクエストが成功すること`

①テストデータとして、ログインユーザーと投稿を生成
```
  let(:user) { create(:user, nickname: 'Takashi') }
  let(:post_instance) { create(:post, user: user, text: 'PostRequestTest', image: 'https://example_image_url') }
```

②subject(テスト対象のオブジェクト)は、投稿一覧のリクエスト処理  
Request specでは直接ルーティング名を指定することができます。
```
subject { get posts_url }
```

③テスト前にテストデータのユーザーでログイン処理とlet変数（post_instance）を呼び出しておく
```
before do
    sign_in user
    post_instance
end
```

④リクストが通ることを確認するので、HTTPレスポンスのステータスが200であることが期待値
```
it_behaves_like 'return_response_status', 200
```



次のテスト内容は以下です。  
`投稿一覧のレスポンスに適切な投稿内容を含んでいること`

①subject(テスト対象のオブジェクト)は先ほどど同じ、投稿一覧のリクエスト処理
```
subject { get posts_url }
```

②返却されたレスポンスのボディに投稿の内容とhtmlを含んでいるか確認します。  
response.bodyの内容を検証しています。
```
expect(response.body).to include 'PostRequestTest :Takashiの投稿'
expect(response.body).to include '<span>投稿者</span>Takashi'
```




上記コード全体

```
  let(:user) { create(:user, nickname: 'Takashi') }
  let(:post_instance) { create(:post, user: user, text: 'PostRequestTest', image: 'https://example_image_url') }

  describe '投稿一覧' do
    subject { get posts_url }

    context 'ログインしている場合' do
      before do
        sign_in user
        post_instance
      end

      it_behaves_like 'return_response_status', 200

      it 'レスポンスに適切な投稿内容を含むこと' do
        subject
        aggregate_failures do
          expect(response.body).to include 'PostRequestTest :Takashiの投稿'
          expect(response.body).to include '<span>投稿者</span>Takashi'
        end
      end
    end
  
  ~~~~省略~~~~~
```





