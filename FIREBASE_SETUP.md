# IDM Hub Firebase Setup

このプロジェクトの `index.html` は、Firebaseをまだ読み込まない静的プロトタイプです。ここでは、あとからFirestoreベースの登録・ログインに差し替えるための手順をまとめます。

## 前提

- Firebase Authenticationは使わない
- ユーザー情報はFirestoreに保存する
- パスワードはBase64文字列として保存する
- GitHub PagesまたはFirebase Hostingで一般公開する

重要: Base64は暗号化ではなく、誰でも復元できるエンコードです。ほかのシステム都合で避けられない場合でも、Firestore Security Rules、HTTPS、アクセス制御、ログ出力抑制を必ず設定してください。

## Firestoreのデータ例

コレクション名の例:

```text
users
```

ドキュメントIDの例:

```text
メールアドレスを正規化した文字列、または自動ID
```

フィールド例:

```js
{
  name: "IDMユーザー",
  email: "user@example.com",
  passwordBase64: "cGFzc3dvcmQxMjM=",
  createdAt: serverTimestamp(),
  updatedAt: serverTimestamp(),
  enabled: true
}
```

## Base64変換の例

日本語や記号を含む可能性を考え、ブラウザ側では次のように変換します。

```js
function toBase64(value) {
  return btoa(unescape(encodeURIComponent(value)));
}
```

復元が必要な場合:

```js
function fromBase64(value) {
  return decodeURIComponent(escape(atob(value)));
}
```

## Firebase SDK追加の流れ

`index.html` の末尾付近、現在の `<script>` をFirebase連携版に変更します。

```html
<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.5/firebase-app.js";
  import {
    getFirestore,
    doc,
    getDoc,
    setDoc,
    serverTimestamp
  } from "https://www.gstatic.com/firebasejs/10.12.5/firebase-firestore.js";

  const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT.appspot.com",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
  };

  const app = initializeApp(firebaseConfig);
  const db = getFirestore(app);
</script>
```

## 登録処理の考え方

```js
async function signup({ name, email, password }) {
  const normalizedEmail = email.trim().toLowerCase();
  const userRef = doc(db, "users", normalizedEmail);
  const existing = await getDoc(userRef);

  if (existing.exists()) {
    throw new Error("このメールアドレスは登録済みです。");
  }

  await setDoc(userRef, {
    name,
    email: normalizedEmail,
    passwordBase64: toBase64(password),
    enabled: true,
    createdAt: serverTimestamp(),
    updatedAt: serverTimestamp()
  });
}
```

## ログイン処理の考え方

```js
async function login({ email, password }) {
  const normalizedEmail = email.trim().toLowerCase();
  const userRef = doc(db, "users", normalizedEmail);
  const snapshot = await getDoc(userRef);

  if (!snapshot.exists()) {
    throw new Error("メールアドレスまたはパスワードが違います。");
  }

  const user = snapshot.data();
  if (!user.enabled || user.passwordBase64 !== toBase64(password)) {
    throw new Error("メールアドレスまたはパスワードが違います。");
  }

  sessionStorage.setItem("idmHubUser", JSON.stringify({
    email: user.email,
    name: user.name
  }));
}
```

ログイン後は、カードの `locked` クラスを外し、実際のリンクを有効化する処理を追加します。

## Firestore Security Rules例

Firebase Authenticationを使わない場合、クライアントからFirestoreへ直接安全に書き込ませるルール設計が難しくなります。最低限の例としては、開発中だけ書き込みを許可し、本番ではCloud Functionsや別サーバー経由に寄せることを推奨します。

開発用の一時ルール例:

```txt
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{email} {
      allow read, write: if false;
    }
  }
}
```

本番でブラウザから直接登録・ログインする場合は、Firebase Authenticationなしでは本人判定ができないため、公開クライアントから全ユーザー情報を読めるようなルールにしないでください。

## Firebase Hosting

Firebase CLIをインストールします。

```bash
npm install -g firebase-tools
firebase login
firebase init hosting
```

設定例:

```text
public directory: .
single-page app: No
automatic builds/deploys with GitHub: 任意
```

公開:

```bash
firebase deploy
```

## GitHub Pages

1. GitHubのリポジトリ `kali-n-coder/IDM-hub` を開く
2. `Settings` を開く
3. `Pages` を開く
4. `Build and deployment` の `Source` を `Deploy from a branch` にする
5. `Branch` を `main`、フォルダを `/root` にする
6. `Save` を押す

数分後に、GitHub PagesのURLが表示されます。

## 次に実装するとよいこと

- 実ツールのURL一覧を `tools` 配列としてJavaScriptに分離する
- ログイン状態の表示をヘッダーに追加する
- Firestore連携時に登録済みユーザーの重複チェックを入れる
- 退会、パスワード変更、利用停止の管理画面を用意する
