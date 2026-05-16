# IDM Hub

便利ツールやミニゲームをまとめる一般公開向けHubサイトです。

## Files

- `index.html`: Firebase HostingとFirestoreに接続済みの公開ページ
- `firestore.rules`: Firestoreのセキュリティルール
- `FIREBASE_SETUP.md`: Firestore連携、GitHub Pages、Firebase Hostingの設定メモ

## Current State

- 一般公開サービス向けのトップページ
- Firestoreを使ったアカウント登録・ログイン
- 未ログイン時のコンテンツロック
- ツール、ゲームの検索とカテゴリフィルター
- ログイン後の外部ツールリンク遷移

## Deploy

Firebase Hostingで公開済みです。詳しい設定は `FIREBASE_SETUP.md` を参照してください。

Firebase Hosting project:

```text
idm-hub-20260516
```

URL:

```text
https://idm-hub-20260516.web.app
```
