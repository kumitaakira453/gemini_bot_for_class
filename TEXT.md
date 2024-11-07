# GeminiBot を作成しよう

## 0 はじめに

今回の授業ではゼロから PJ を作成する時間がないので、ある程度 PJ が出来上がっているものを
ベースに完成させていくことにします。
PJ を作成するフォルダを開いて以下を実行してください。

```shell
    $ git clone https://github.com/kumitaakira453/gemini_bot_for_class.git
```

クローンするとフォルダ内に PJ が作成されます。

```shell
    gemini_bot_for_class
    ├── README.md
    ├── TEXT.md
    ├── assets
    │   └── gemini_logo.png
    ├── index.html
    ├── src
    │   └── main.js
    └── styles
        └── index.css
```

簡単にフォルダ構造を確認しておきましょう。

- **index.html**:　メインとなる画面の HTML、必要に応じて JS で内容を操作する
- **src/javascript**: JS ファイルをまとめている
  - **src/main.js**:今回の授業で主に扱う、ページの全体的な処理をまとめている
- **assets/**：画像ファイルをまとめている
- **styles/css**: CSS ファイルをまとめている

フォルダ分けは、ある程度 PJ が大きくなって扱うファイルが増えた際に、プロジェクトの可読性や管理の効率を高めるために非常に重要です。適切なフォルダ構造を作成することで、チームメンバー間での作業分担や、ファイルの検索・管理が容易になります。

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
   コードを追加することで、`index.html`ファイルが読み込まれると同時に、処理が実行されます。

   ```javascript
   // 初期化
   document.addEventListener("DOMContentLoaded", () => {
     // 以下はコメントアウトしておく
     // const messageManager = new MessageManager();
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

1. 送信ボタンにイベントを追加する

   1. まず、`DOMContentLoaded`イベントのコールバック関数内の`MessageManager`クラスのインスタンス化の部分を元に戻し、
      逆に先ほど追加した`sendToGemini`の関数の呼び出し部分は不要なのでコメントアウトしておいてください。

   ```javascript
   const messageManager = new MessageManager();
   // 1.5.3 geminiにリクエストを送信する関数を作成する
   // sendToGemini("こんにちは");
   ```

   2. 次に`sendBtn`がクリックされたときに先ほど説明した`sendMessage()`が実行されるように、コードを追加してください。
      既存コードの処理ごとの記述位置や記述方法を参考にしてください。

2. HTML 要素を複製して追加する

   1. HTML 上に新しい HTML 要素を追加する方法としては、大きく分けて
      1. JS コード上で HTML 要素を定義して追加する
      2. 既存の HTML 要素をコピー・編集して追加する

   の 2 つがあります。前者は比較的簡単な構造を持った HTML 要素を追加する際に JS のみで処理を記述できる点で便利ですが、複雑な構造の
   HTML 要素を追加する際には記述が多くなってしまいコードの可読性が下がるというデメリットもあります。今回はそこまで複雑な HTML 要素とは言えませんが、
   各要素に対してクラスの指定が必要な点などから、後者の方法を選択することお勧めします。

   2. 今回コピーする HTML 要素はそれぞれ`index.html`上にある
      `<template>`タグ内にあらかじめ書いてあります。(`<template>`タグは画面上に描画されないタグで、後から JS などで追加する HTML 要素の骨格をあらかじめ用意
      しておくために使われるタグです。)
      1. 今回 HTML 上には二つの template が用意されています。
         1. `#message_template_from_me`：自分から送信したメッセージを表示する template を持っている
         2. `#message_template_from_friend`：相手から(gemini)からのメッセージを表示する template を持っている
            いずれも`<template>`タグ内の`div.message`タグをコピーして HTML 上に追加します。
   3. まずは、自分が送信したメッセージを HTML 上に追加するコードを書いてみましょう。メッセージが送信された際に呼び出される`sendMessage`関数内で実行されている`createMessage`関数に処理を記述してみましょう。コードを書く際には以下の点に注意してください。

      1. `<template>`をコピーした上で、適切に HTML 要素を編集する必要があります。引数として受け取った以下の情報を追加してください。
         1. メッセージ本文(`content`)
         2. 送信時間(`time`)
      2. 関数の引数のうち`isFromMe`はこの後で Gemini からのメッセージを追加する際に関数の処理を一部変更するために使います。現時点では特に考える必要はありません。

   4. 以下にコードの例を示しています。適宜活用してください。

      ```javascript
      createMessage(content, time, isFromMe = true) {
           const messageTemplate = document.querySelector(
               `#message_template_from_me`
           );
           const message = messageTemplate.content
               .cloneNode(true)
               .querySelector(".message");
           const sendTimeTag = message.querySelector(".message__time");
           const contentTag = message.querySelector(".message__content");

           sendTimeTag.textContent = time;
           contentTag.textContent = contentHTML;
           this.messageContainer.appendChild(message);
           this.scrollToBottom(message);
       }
      ```

   5. 作成したコードが動くか確認しましょう。第 2 引数の`time`を取得する関数はまだ作成していないので、適当な引数を追加しています。

      ```javascript
       async sendMessage() {
           const content = this.sendBox.value.trim();
           if (content !== "") {
               // コメントアウトしておく
               // const time = this.getFormattedTime();
               this.createMessage(content, '20:22');
               this.sendBox.value = "";
               this.toggleMicSendButton();
               // コメントアウトしておく
               // await this.sendToGemini(content);
           }
       }
      ```

3. Gemini と API 通信し、レスポンスを HTML 上に追加する
   では次に、先ほど作成した Gemini と API 通信するコードを元に、Gemini にメッセージを送り
   その返信を HTML 上にメッセージとして追加する処理を書いていきましょう。
   1. `sendToGemini`関数で、Gemini とのやりとり、メッセージの描画を行います。メッセージ描画については
      先ほど作成した`createMessage`関数を再利用しましょう。
   2. 完成した`sendToGemini`関数の一例です。適宜参考にしてください。`isFromMe`引数に`false`を設定する
      必要があるので忘れないようにしてください。
      ```javascript
       async sendToGemini(message) {
           const genAI = new GoogleGenerativeAI(APIKEY);
           const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });
           const result = await model.generateContent(message);
           this.createMessage(
               result.response.text(),
               //時刻を取得する関数はまだ作成していないので適当な時間を入れておく。
               '20:23',
               false
           );
       }
      ```
4. 前述のように、Gemini からのメッセージも描画できるように`createMessage`を編集する必要があります。以下の点に注意して編集してみてください。

   1. メッセージが自分からのものなのか、Gemini からのものなのかは`isFromMe`からわかる。
   2. 送信元に応じて使うテンプレートを変更する必要がある。

5. コード例は以下です。適宜参照してください。

   ```javascript
   createMessage(content, time, isFromMe = true) {
     const templateName = isFromMe ? "me" : "friend";
     const messageTemplate = document.querySelector(
         `#message_template_from_${templateName}`
     );
     const message = messageTemplate.content
         .cloneNode(true)
         .querySelector(".message");
     const sendTimeTag = message.querySelector(".message__time");
     const contentTag = message.querySelector(".message__content");

     sendTimeTag.textContent = time;
     contentTag.textContent = content;
     this.messageContainer.appendChild(message);
     this.scrollToBottom(message);
   }
   ```

6. ここまでかけたら、動くか確認してみましょう。
7. 次に、送信時間(実行時の**現在時刻**)を取得する関数を作成しましょう。JS では`Date`クラスを用いることで
   比較的簡単に時刻情報を取得、作成することができます。以下は現在時刻を取得する例です。

   ```javascript
   const now = new Date();
   ```

   ただし、今回は取得した現在時刻を'12:04'のような形式にする必要があります。[このサイト](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Date)を参考にしながら、
   現在時刻を適切に format した時刻を返却する関数を作成してください。
   以下はコード例です。適宜参考にしてください。

   ```javascript
    getFormattedTime() {
        const now = new Date();
        const hours = now.getHours().toString().padStart(2, "0");
        const minutes = now.getMinutes().toString().padStart(2, "0");
        return `${hours}:${minutes}`;
    }
   ```

8. Gemini からのメッセージをみると、所々おかしな記号が含まれていることが確認できると思います。
   Gemini からの返信は**ほぼ**`markdown`形式なので、これを`HTML`形式で解釈(変換)してやる必要があります。
   実装にあたってはこのような機能を提供してくれる[`Marked.js`](https://github.com/markedjs/marked)というパッケージを使います。

   1. まず、HTML 上でこのパッケージを使えるようにしましょう。`index.html`の`<head>`内に以下を追加してください。
      ```html
      <!-- markdownに対応 -->
      <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
      ```
   2. 使い方は非常に簡単で、`marked.parse()`関数に`markdown`形式のテキストを引数として渡すだけです。
      ```javascript
      const mdContent = "##markdownで記述されたテキスト";
      const htmlContent = marked.parse(mdContent);
      ```
   3. では実装していきましょう。`createMessage`関数を編集して`Gemini`から受け取ったメッセージを`HTML`に
      変換してから HTML 要素に追加するように編集しましょう。
   4. 以下が実装例になります。HTML 要素を追加するので`innerHTML`を使っています。

   ```javascript
    createMessage(content, time, isFromMe = true) {
        const templateName = isFromMe ? "me" : "friend";
        const messageTemplate = document.querySelector(
            `#message_template_from_${templateName}`
        );
        const message = messageTemplate.content
            .cloneNode(true)
            .querySelector(".message");
        const sendTimeTag = message.querySelector(".message__time");
        const contentTag = message.querySelector(".message__content");

        sendTimeTag.textContent = time;
        let contentHTML = content;
        if (!isFromMe) {
          // HTML要素全体をまとめてる親タグを作成し外余白を与えている。
            contentHTML = `<div style="margin:5px 5px;">${marked.parse(
                content
            )}</div>`;
        }
        // innerHTMLを用いる
        contentTag.innerHTML = contentHTML;
        this.messageContainer.appendChild(message);
        this.scrollToBottom(message);
    }
   ```

## 3 　追加実装(時間が余ったら)

せっかくなので、そのほかの API も Bot に追加してみましょう。以下に簡単に叩ける 面白そうな API の例を示しておきます。
使用例はそれぞれのサイトをみると書かれているので、ぜひ自力で実装してみてください。
| API | 概要 | 用途 | URL |
|------------------------------|----------------------------------------------|-------------------------------------------|--------------------------------------------|
| Dog CEO's Dog API | ランダムな犬の画像や犬種ごとの画像を取得 | 犬画像を表示するサイトやアプリ | [https://thedogapi.com](https://thedogapi.com) |
| The Cat API | ランダムな猫の画像や猫種の情報を提供 | 猫画像を表示するサイトやアプリ | [https://thecatapi.com](https://thecatapi.com) |
| OpenWeatherMap API | 世界中の天気情報を取得 | 天気予報アプリや気温表示ウィジェット | [https://openweathermap.org/api](https://openweathermap.org/api) |
| NASA APOD API | NASA が公開する「今日の宇宙写真」を取得 | 宇宙や天文学に関するアプリ | [https://api.nasa.gov](https://api.nasa.gov) |
| Advice Slip API | ランダムなアドバイスを提供 | おみくじアプリやモチベーションサイト | [https://api.adviceslip.com](https://api.adviceslip.com) |
| JokeAPI | ジョークやユーモアのあるテキストを提供 | 楽しみや娯楽のアプリ、ランダムジョークボタン | [https://v2.jokeapi.dev](https://v2.jokeapi.dev) |
| PokéAPI | ポケモンの詳細情報や画像を取得 | ポケモン好き向けのアプリやゲーム | [https://pokeapi.co](https://pokeapi.co) |
| Number Facts API | 数字に関するランダムな雑学情報を提供 | 数学のクイズや雑学を提供するアプリ | [http://numbersapi.com](http://numbersapi.com) |

## 4 JS の基本文法(確認用)

JS を用いた基本的な DOM(`Document Object Mode`) 操作についてまとめておきます。適宜確認に使ってください。

### 4.1 要素の取得

HTML 上の要素を取得する方法としては以下のようなものがあります。

- **getElementById**

  - id 属性を指定して要素を取得する

    ```javascript
    const element = document.getElementById("myId");
    ```

- **getElementsByClassName**

  - クラス名を指定して要素を取得する。戻り値はリスト形式。

    ```javascript
    const elements = document.getElementsByClassName("myClass");
    ```

- **querySelector/querySelectorAll**

  - CSS セレクタを指定して、要素を取得する。`querySelector`は最初の要素のみ、`querySelectorAll`
    はすべての一致する要素を返す。

    ```javascript
    const element = document.querySelector(".myClass"); // 最初の一致する要素
    const elements = document.querySelectorAll(".myClass"); // すべての一致する要素
    ```

### 4.2 要素の編集

- **`textContent`**

  - 要素のテキストを編集する。

    ```javascript
    const element = document.getElementById("myId");
    element.textContent = "新しいテキスト";
    ```

- **`innerHTML`**

  - 要素の HTML を編集する。

    ```javascript
    element.innerHTML = "<span>新しいHTML内容</span>";
    ```

- **`setAttribute`**

  - 要素の属性を変更するメソッド
    ```javascript
    element.setAttribute("src", "new-image.png");
    ```
  - 直接プロパティを指定することもできます。

    ```javascript
    element.href = "https://example.com";
    ```

  - `style`属性を用いることで要素の CSS を操作することもできます。
    ```javascript
    element.href.color = "blue";
    ```

### 4.3 要素の追加

#### 要素の作成

`createElement`で新しい要素を作成できます。

```javascript
const newElement = document.createElement("div");
newElement.textContent = "追加された要素";
```

#### 要素の挿入

作成した HTML 要素を追加するには、`appendChild`を用います。

```javascript
const parent = document.getElementById("parentElement");
parent.appendChild(newElement);
```

その他の挿入方法としては以下のようなものがあります。

- `prepend`: 親要素の最初に挿入
- `before`: 指定した要素の前に挿入
- `after`: 指定した要素の後に挿入

#### 要素の削除

`cloneNode`メソッドで要素を複製できる。

- 引数に `true` を渡すと、子要素も含めて複製（深いコピー）。
- 引数に `false` を渡すか、省略すると要素自体のみ（浅いコピー）。

```javascript
const originalElement = document.getElementById("original");
const shallowCopy = originalElement.cloneNode(false); // 子要素は複製しない
const deepCopy = originalElement.cloneNode(true); // 子要素も含めて複製
parentElement.appendChild(deepCopy); // 複製した要素を追加
```
