# システムテスト(ST)
## 本セクションで行うこと
1. システムテストについて
2. System spec解説


## システムテストについて

システムテスト(System Test)は、要件定義書に記載した要件をクリアしているか?、つまりアプリ全体が一つのシステムとして期待される動きをするのか?、　　
を確認するテストになります。  
これまで説明した単体テスト、結合テストとは、詳細設計、基本設計で記載した要件をクリアしているかどうかを確認していました。  
このような形のテストモデルを「V字モデル」と呼びます。

<img width="648" alt="スクリーンショット 2022-03-21 15 34 10" src="https://user-images.githubusercontent.com/52161269/159214576-b1c97d34-531e-42b1-a467-2461a0dc8bfe.png">

では、実際にシステムテストを行う際はどのようにテストケースを実施すれば良いでしょうか?  
システムテストの手法としていくつかありますが、今回はシナリオテストを行います。

シナリオテストは、実際のユーザーの行動をシナリオ化してテストを行います。  
投稿機能を例に考えてみましょう。  

```
投稿機能におけるシステムテスト：
1. ユーザーは投稿する
2. ユーザーは投稿の編集をする
3. ユーザーは投稿の削除をする
4. ユーザーは自分の投稿にコメントをする
5. ユーザーは他人の投稿を閲覧し、コメントをする
6. ...以下省略...
```

## System spec解説

Rpsecでは、システムテストとして行うテストをSystem specと呼んでいます。
System specでは、ユーザーの動きをコードに起こしてテストする為に、「Capybara」と言うgem(ライブラリ）を使います。  
Capybaraは、Rails5.1以降では標準インストールされています。  
Capybaraは、指定したページにアクセスしたり、フィールドに入力したり、ボタンをクリックしたり、ブラウザの操作を可能にします。


先ほどの投稿機能のシステムテストケースを使って、実際のコードを確認していきましょう！

テスト内容：`1. ユーザーは投稿する`

①投稿する為にユーザーを用意
```
let(:user_a) { create(:user, nickname: 'Takashi') }
```

②テスト実施前に、ユーザーのログイン処理を行う
```
before do
    sign_in_as user_a
  end
```

③テスト対象は`ユーザーは投稿する`なので、Capybaraのメソッドでブラウザ操作して投稿処理をする  
```
expect {
        click_link_or_button '投稿する'
        fill_in 'post_text', with: 'sample_text'
        click_button 'commit'

        post = Post.find_by_text_and_user_id('sample_text :Takashiの投稿', user_a.id)

        aggregate_failures do
          expect(current_path).to eq posts_path
          expect(page).to have_content 'sample_text'
          expect {
            SlackSyncJob.perform_later(post)
          }.to have_enqueued_job.with(post)
        end
      }
```

**Capybaraメソッド**  
`click_link_or_button　'ボタン名orID名'`・・・文字列で指定したボタンもしくはリンクを探してクリックする。  
`fill_in '', with: ''`・・・













