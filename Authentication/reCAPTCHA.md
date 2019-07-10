### reCAPTCHA v3
- 閲覧者のページ内の行動からスコアを算出し、bot を判別する (画像を選択するなどのユーザの操作が不要となる)。
- v2 ではサイトキーから自動的にトークンを取得してくれたが、v3 では取得の部分から実装しなければならない
    - (※ トークン取得については、gem を使用することで実装しなくても良い)
- 取得したトークンの有効期限は２分のため、トークンを取得するタイミングも考慮しなければならない。
    - フォームを submit する手前とか

- bot かどうかの検証 API を実行すると以下のレスポンスが返却される。

```
{
  "success": true|false,      // whether this request was a valid reCAPTCHA token for your site
  "score": number             // the score for this request (0.0 - 1.0)
  "action": string            // the action name for this request (important to verify)
  "challenge_ts": timestamp,  // timestamp of the challenge load (ISO format yyyy-MM-dd'T'HH:mm:ssZZ)
  "hostname": string,         // the hostname of the site where the reCAPTCHA was solved
  "error-codes": [...]        // optional
}
```

gem を使用すると上記レスポンスを検証し boolean を返却するため、上記属性を使用する用途が出てきた場合 gem を使用せずに自前で実装する必要がありそう。
[自前実装する場合の参考資料](https://www.techscore.com/blog/2018/12/06/recaptcha-v3/)

## 手法
- あらかじめ、[管理コンソール](https://www.google.com/recaptcha/admin/create) ページにてサイトキーとシークレットキーを取得する。

- gem: [recaptcha](https://github.com/ambethia/recaptcha)
    - スターが断トツで多い
    - 5月中旬に v3 に対応
    - [verify_recaptcha](https://github.com/ambethia/recaptcha#verify_recaptcha-use-with-v3) を使用し、ユーザの行動に基づくスコアと照らし合わせ bot であるか否かの [boolean 値](https://github.com/ambethia/recaptcha/blob/c9ef2815a3c3d7b8e0abd4b5b85207a34db23cc0/lib/recaptcha/adapters/controller_methods.rb#L27)が返却される
        - スコア値が 0.5 未満だった場合、bot と判断しそれ以降の処理を捌く (0.5 という数値は Google が定める人と bot の閾値)
        - なお、`verify_recaptcha` の引数として閾値を設定することも可能

- gem: [new_google_recaptcha](https://github.com/igorkasyanchuk/new_google_recaptcha)
    - スターが少ない
    - v3 に対応
    - [human?](https://github.com/igorkasyanchuk/new_google_recaptcha/blob/b334ffca19dfa88e887da80de02af9280e965768/lib/new_google_recaptcha.rb#L12) を使用し、bot であるか否かを boolean 値で返却する
    - 上記 gem と使い方もさほど変わらない


## 参考資料
- [reCAPTCHA v3 公式ドキュメント](https://developers.google.com/recaptcha/docs/v3)
