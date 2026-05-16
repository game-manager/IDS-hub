# IDS Hub

便利ツールやミニゲームをまとめる一般公開向けHubサイトです。

## Files

- `index.html`: GitHub Pagesで公開するトップページ
- `admin.html`: ユーザーとツールを管理する画面
- `firestore.rules`: Firestoreのセキュリティルール
- `FIREBASE_SETUP.md`: Firestore連携とGitHub Pagesの設定メモ

## Current State

- 一般公開サービス向けのトップページ
- Firestoreを使ったアカウント登録・ログイン
- 未ログイン時のコンテンツロック
- Firestore上のツール、ゲームの検索とカテゴリフィルター
- ログイン後の外部ツールリンク遷移
- 管理画面からのユーザー管理、ツール管理

## Deploy

GitHub Pagesで公開済みです。FirebaseはFirestoreだけを使っています。

GitHub Pages:

```text
https://kali-n-coder.github.io/IDS-hub/
```

Admin:

```text
https://kali-n-coder.github.io/IDS-hub/admin.html
```

Firebase project:

```text
idm-hub-20260516
```

Firebase Hostingは停止済みです。
