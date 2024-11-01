## 実行方法

### 1.Gemini の APIKEY を取得する

1. [Gemini Developer API](https://ai.google.dev/)に開くアクセス
2. 画面中央にある「Get API key in Google AI Studio」をクリック
3. 「法的通知」を読んだ上でチェックボックスにチェックを入れて「続行」をクリック
4. 「API キー」を取得から「API を作成」ボタンをクリック
5. 確認モーダルが開くので「OK」をクリック
6. 「新しいプロジェクトで API キーを作成」を選択
   1. GCP 等が開くことはなく、PJ が自動で作成されるだけ
7. しばらくすると「API キーが生成されました」というメッセージの下に KEY が生成されているのが確認できる

### 2.APIKEY を.env に追加する

※正しい環境変数の運用とは言えないですが、最低限 github 上にあげないための対応として行なっています。

1. `src`フォルダ内に`env.js`というファイルを追加し、以下のように記載する

   ```javascript:src/env.js
   export const APIKEY = "[取得したAPIKEY]";
   ```
