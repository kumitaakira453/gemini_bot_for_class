# GeminiBot を作成しよう

## 1 GeminiAPI の導入

### 1.1 API とは

API（Application Programming Interface）は、異なるソフトウェアやサービスが相互に通信するための仕組みや規約を提供するインターフェースのことです。API により、開発者は他のサービスやアプリケーションの機能にアクセスし、利用できるようになります。

たとえば、天気情報を提供する API を利用すると、アプリケーションはその API を通してリアルタイムの天気データを取得し、ユーザーに表示できます。API はリクエストとレスポンスの形式を定めることで、情報の交換を標準化し、異なるシステム間のやりとりを効率的に行う手助けをしています。

API の主な利点は次のとおりです：

- **再利用性**：既存の機能を再利用でき、アプリケーションを短期間で開発できる。
- **相互運用性**：異なるシステム間で情報をやり取りできる。
- **抽象化**：複雑な機能を隠蔽し、簡単に使えるようにする。

### 1.2 GeminiAPI とは

Gemini API は、Google が提供する最新の生成型 AI モデル「Gemini」へのアクセスを提供する API です。Gemini は大規模な自然言語処理モデルで、ユーザーの入力に対して文章の生成や質問応答、要約などの高度なタスクを実行できます。Google の Gemini モデルは、従来の自然言語処理能力を進化させたものであり、会話の文脈を理解し、ユーザーのニーズに応じたより洗練された応答を提供することが可能です。

Gemini API を使用すると、開発者は以下のような機能をアプリケーションに組み込むことができます：

- **会話生成**：ユーザーの入力に応じた自然な会話の生成。
- **質問応答**：知識ベースをもとにした質問応答。
- **テキスト要約**：長文を簡潔に要約。
- **クリエイティブなテキスト生成**：詩や物語の生成など、創造的な応答。
  Gemini API は、開発者が高度な AI 機能をアプリケーションに統合するためのアクセス手段を提供し、さまざまな業界やサービスでの応用が期待されています。

### 1.3 APIKey の取得

外部 API を使うためには、APIKey を取得する必要があります。以下の手順で Gemini の APIKey を取得してください。

1. [Gemini Developer API](https://ai.google.dev/)にアクセス
2. 画面中央にある「Get API key in Google AI Studio」をクリック
3. 「法的通知」を読んだ上でチェックボックスにチェックを入れて「続行」をクリック
4. 「API キー」を取得から「API を作成」ボタンをクリック
5. 確認モーダルが開くので「OK」をクリック
6. 「新しいプロジェクトで API キーを作成」を選択
   1. GCP 等が開くことはなく、PJ が自動で作成されるだけ
7. しばらくすると「API キーが生成されました」というメッセージの下に KEY が生成されているのが確認できます。

### 1.4 APIKEY を.env に追加する

取得した APIKey を今回作成する PJ に追加します。

1. `src`フォルダ内に`env.js`というファイルを追加し、以下のように記載する

   ```javascript:src/env.js
   export const APIKEY = "[取得したAPIKEY]";
   ```

### 1.5 API を使うための設定

GeminiAPI には、API を簡単に利用できるようにパッケージが提供されています。
まずは、PJ にパッケージを導入しましょう。

1. `index.html`の`<head>`タグ内に以下を追加

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="ja">
  <head>
    #...
    <!-- 1.5.1パッケージの導入 -->
    <script type="importmap">
      {
        "imports": {
          "@google/generative-ai": "https://esm.run/@google/generative-ai"
        }
      }
    </script>
  </head>
</html>
```

2. `src/main.js`の一番上に以下を追加する

```javascript
// 1.5.1 パッケージの追加
import { GoogleGenerativeAI } from "@google/generative-ai";
import { APIKEY } from "./env.js";
#...
```

3. `src/main.js`に Gemini にリクエストを送信する関数を作成する
   Gemini にリクエストを送信して、gemini からのレスポンスを console に表示する関数を作成しましょう。
   API に対してリクエストを送信し、そのレスポンスを待つ処理はすぐさま終了するものではありません。通常 API 通信を含む処理は他のプログラムの処理時比べて長い時間がかかります。そのような場合は、通常通り次の処理にすぐさま移行してしまうと、API からのレスポンスを受け取ることができません。このような時には`非同期処理`というものを用います。
   ここで詳しい説明は避けますが、簡単に説明すると、レスポンスが帰ってくるまで次の処理に進まず待機し、レスポンスが帰ってきたら次の処理に進むというものです。
   以下のようなコードを作成してみてください。

   ```javascript
   // 1.5.3 geminiにリクエストを送信する関数を作成する
   async function sendToGemini(message) {
     const genAI = new GoogleGenerativeAI(APIKEY);
     const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });
     const result = await model.generateContent(message);
     console.log(result.response.text());
     return result.response.text();
   }
   ```

   実行に時間がかかる処理の前に`await`というキーワードをつけることで処理を待つことが
   できます。また、`await`を用いる際は関数の最初に`async`というキーワードをつける
   必要があるので、セットで覚えておいてください。

   では実際に関数を呼び出してみましょう。
   `src/main.js`の`DOMContentLoaded`イベントのコールバック関数内に以下のように
   コードを追加することで、`index.html`ファイルが読み込まれると同時に、処理が実行される。

   ```javascript
   // 初期化
   document.addEventListener("DOMContentLoaded", () => {
     // 1.5.3 geminiにリクエストを送信する関数を作成する
     sendToGemini("こんにちは");
   });
   ```

   ここまで書けたら vscode の拡張機能である「LiveServer」を利用してローカルサーバーを立ち上げ、指定された PORT にアクセスしてください。正しく実行できると、コンソールにレスポンスの文章が表示されます。

## 2 メッセージ送信時の画面処理を作成する

### 2.1 コードの全体像を確認する

`src/main.js`のコードは以下のような構造になっています。

```javascript
class MessageManager {
  constructor() {
    this.initializeElements();
    this.isComposing = false;
    this.initializeEvents();
    document.querySelector(".message__time#now_time").innerHTML =
      this.getFormattedTime();
  }

  /**
   * クラスで使用するDOM要素を初期化する
   */
  initializeElements() {
    this.sendBox = document.querySelector("#input");
    this.messageContainer = document.querySelector("#main_container");
    this.addBtn = document.querySelector("#add_btn");
    this.cameraBtn = document.querySelector("#camera_btn");
    this.imageBtn = document.querySelector("#image_btn");
    this.arrowBtn = document.querySelector("#arrow_btn");
    this.micBtn = document.querySelector("#mic_btn");
    this.sendBtn = document.querySelector("#send_btn");
  }

  /**
   * 各UI要素にイベントリスナーを設定する
   */
  initializeEvents() {
    this.sendBox.addEventListener("input", () => this.toggleMicSendButton());
    this.sendBox.addEventListener(
      "compositionstart",
      () => (this.isComposing = true)
    );
    this.sendBox.addEventListener(
      "compositionend",
      () => (this.isComposing = false)
    );
    this.sendBox.addEventListener("keydown", (e) => this.handleEnterKey(e));
    this.sendBox.addEventListener("focus", () => this.toggleInputLeftBtn(true));
    this.sendBox.addEventListener("blur", () => this.toggleInputLeftBtn(false));
  }

  /**
   * メッセージコンテナをボトムまでスクロールする
   * @param {HTMLElement} target - スクロール対象の要素
   */
  scrollToBottom(target) {
    this.messageContainer.scrollTop = this.messageContainer.scrollHeight;
    const targetPosition = target.offsetTop;
    const offset = window.innerHeight * 0.3;
    this.messageContainer.scrollTop = targetPosition - offset;
  }

  /**
   * 入力ボックスの値に応じてマイクボタンと送信ボタンの表示を切り替える
   */
  toggleMicSendButton() {
    const sendBoxValue = this.sendBox.value;
    if (sendBoxValue.trim() === "") {
      this.micBtn.classList.remove("hidden");
      this.sendBtn.classList.add("hidden");
    } else {
      this.micBtn.classList.add("hidden");
      this.sendBtn.classList.remove("hidden");
    }
  }

  /**
   * 入力ボックスのフォーカス状態に応じてボタンの表示を切り替える
   * @param {boolean} isFocused - 入力ボックスがフォーカスされているかどうか
   */
  toggleInputLeftBtn(isFocused) {
    const notFocusBtns = [this.addBtn, this.cameraBtn, this.imageBtn];
    if (isFocused) {
      notFocusBtns.forEach((btn) => {
        btn.classList.add("hidden");
      });
      this.arrowBtn.classList.remove("hidden");
    } else {
      notFocusBtns.forEach((btn) => {
        btn.classList.remove("hidden");
      });
      this.arrowBtn.classList.add("hidden");
    }
  }

  /**
   * Enterキーが押された際にメッセージを送信する
   * @param {KeyboardEvent} e - キーボードイベント
   */
  handleEnterKey(e) {
    if (e.key === "Enter" && !this.isComposing) {
      e.preventDefault();
      this.sendMessage();
    }
  }

  /**
   * メッセージ内容をGemini AIモデルに送信し、レスポンスメッセージを作成する
   * @param {string} message - 送信するメッセージ内容
   */
  async sendToGemini(message) {}

  /**
   * 入力ボックスに入力されたメッセージを送信する
   */
  async sendMessage() {
    const content = this.sendBox.value.trim();
    if (content !== "") {
      const time = this.getFormattedTime();
      this.createMessage(content, time);
      this.sendBox.value = "";
      this.toggleMicSendButton();
      await this.sendToGemini(content);
    }
  }

  /**
   * 現在の時刻をHH:MM形式で返す
   * @returns {string} フォーマットされた時刻文字列
   */
  getFormattedTime() {}

  /**
   * 新しいメッセージを作成し、メッセージコンテナに追加する
   * @param {string} content - メッセージ内容
   * @param {string} time - メッセージの送信時刻
   * @param {boolean} [isFromMe=true] - メッセージがユーザーからのものであるかどうか
   */
  createMessage(content, time, isFromMe = true) {}
}

// 初期化
document.addEventListener("DOMContentLoaded", () => {
  const messageManager = new MessageManager();
});
```

細かな処理を行う関数が書かれてはいますが、全体像としては比較的シンプルです。
実行順に構造を確認してみましょう。

1. `index.html`ファイルが読み込まれると同時に、ページ全体の処理をまとめておく`MessageManager`クラスを、インスタンス化します。
2. `MessageManager`がインスタンス化される際に呼び出されるのが、`constructor`関数です。python の`__init__`メソッドにあたります。
3. `constructor`でまず呼び出されている関数が`initializeElements`です。(クラスメソッドを呼び出す際に関数名の前についている`this`は python では`self`と記述される、インスタンス自体を示すキーワードです)`initializeElements`では、html 上の JS の操作対象となる html 要素です。インスタンス化時にクラス変数として取得しておくことで、クラス全体で使用することができます。
4. 次に呼び出されるのが、`initializeEvents`です。これはクラス変数として取得したそれぞれの要素に対してクリックなどの`イベント`が起きたタイミングで呼び出されるコールバック変数を登録する関数です。
5. また、この後メッセージ送信時に作成する関数として以下のような関数を用意しています。
   1. `sendMessage`:「送信」ボタンが押された際に実行される関数
   2. `sendToGemini`:Gemini との API 通信をする関数
   3. `createMessage`:メッセージを HTML 上に追加する関数
   4. `getFormattedTime`:`createMessage`ないで呼びだされる送信時間を取得する関数

### 2.2 送信したメッセージを表示する
