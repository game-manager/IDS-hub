# IDS Hub Firebase Setup

このプロジェクトの `index.html` は、GitHub Pagesで公開し、Firebase Firestoreに接続しています。登録時にFirestoreへユーザーを作成し、ログイン時に `passwordBase64` を照合します。

## 作成済みFirebaseプロジェクト

CodexからFirebase CLIを使い、新規Firebaseプロジェクトを作成済みです。

```text
Project ID: idm-hub-20260516
Project Name: IDS Hub
Console: https://console.firebase.google.com/project/idm-hub-20260516/overview
```

Note: `Project ID` は作成済み識別子のため `idm-hub-20260516` のままです。Firebase Console上の表示名とWeb App表示名は `IDS Hub` に修正済みです。

このリポジトリには、Firestore Rules用に以下のファイルを追加しています。

```text
.firebaserc
firebase.json
firestore.rules
```

公開URL:

```text
https://kali-n-coder.github.io/IDS-hub/
```

管理画面:

```text
https://kali-n-coder.github.io/IDS-hub/admin.html
```

Firebase Hostingは停止済みです。再度使う必要が出た場合だけ `firebase deploy --only hosting` を実行してください。

## 前提

- Firebase Authenticationは使わない
- ユーザー情報はFirestoreに保存する
- パスワードはBase64文字列として保存する
- GitHub Pagesで一般公開する

重要: Base64は暗号化ではなく、誰でも復元できるエンコードです。ほかのシステム都合で避けられない場合でも、Firestore Security Rules、HTTPS、アクセス制御、ログ出力抑制を必ず設定してください。

## Firestoreのデータ例

コレクション名の例:

```text
users
```

ドキュメントID:

```text
小文字化したメールアドレス
```

フィールド例:

```js
{
  name: "IDSユーザー",
  email: "user@example.com",
  passwordBase64: "cGFzc3dvcmQxMjM=",
  enabled: true,
  createdAt: serverTimestamp(),
  updatedAt: serverTimestamp(),
  lastLoginAt: serverTimestamp()
}
```

ツールは `tools` コレクションで管理します。管理画面から追加・編集・停止・削除できます。

```js
{
  name: "JSON Formatter",
  description: "JSONの整形、圧縮、簡易バリデーションを行うツールです。",
  url: "https://jsonformatter.org/",
  category: "tool dev",
  label: "開発ツール",
  color: "dark",
  enabled: true,
  order: 1,
  updatedAt: serverTimestamp()
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

## Firebase SDK設定

`index.html` には次のFirebase Webアプリ設定を組み込み済みです。

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
    projectId: "idm-hub-20260516",
    appId: "1:315082566768:web:03c7c4b57f75ef9a01d8b5",
    storageBucket: "idm-hub-20260516.firebasestorage.app",
    apiKey: "AIzaSyC_nT_seXtKnZWff-76jvJT0EZfkL21TxE",
    authDomain: "idm-hub-20260516.firebaseapp.com",
    messagingSenderId: "315082566768"
  };

  const app = initializeApp(firebaseConfig);
  const db = getFirestore(app);
</script>
```

## 登録処理

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

## ログイン処理

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

  sessionStorage.setItem("idsHubUser", JSON.stringify({
    email: user.email,
    name: user.name
  }));
}
```

ログイン後は、カードの `locked` クラスを外し、実際のリンクを有効化する処理を追加します。

## Firestore Security Rules

`firestore.rules` をデプロイ済みです。

```bash
firebase deploy --only firestore:rules --project idm-hub-20260516
```

管理画面追加に伴い、現在のルールではブラウザから `users` と `tools` の管理操作ができるようにしています。Firebase AuthenticationなしではFirestore Rulesだけで管理者本人を安全に判定できないため、この構成はプロトタイプ用途です。本番運用前に、管理操作はCloud Functionsなどサーバー側に移してください。

## GitHub Pages

現リポジトリ名は `kali-n-coder/IDS-hub` です。

1. GitHubのリポジトリ `kali-n-coder/IDS-hub` を開く
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
