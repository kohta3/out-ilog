# outilog.com ホスティング（招待ディープリンク用）

招待リンク `https://outilog.com/invite?code=XXXX` を機能させるために、
`outilog.com` を配信している **GitHub Pages リポジトリ** にこのフォルダの中身をコピーする。

## 配置（outilog.com のサイトリポジトリのルートに置く）
```
/.well-known/apple-app-site-association   # iOS Universal Links（拡張子なし）
/.well-known/assetlinks.json              # Android App Links
/invite/index.html                        # 受け皿ページ（アプリ有→起動 / 無→ストア）
/.nojekyll                                # ← 重要（下記）
```

## GitHub Pages の注意点（ハマりどころ）
- **Jekyll は `.well-known` のようなドット始まりフォルダを無視する。** ルートに空の **`.nojekyll`** を置いて Jekyll を無効化すること（同梱済み）。
- AASA は**拡張子なし**のファイル名 `apple-app-site-association` のまま置く。リダイレクトなしの 200 で配信されること。
- 配信後に確認:
  ```bash
  curl -sI https://outilog.com/.well-known/apple-app-site-association   # 200, リダイレクトなし
  curl -s  https://outilog.com/.well-known/assetlinks.json
  curl -sI https://outilog.com/invite?code=TEST                          # 200
  ```

## 要差し替え（プレースホルダ）
- `assetlinks.json` の `REPLACE_WITH_PLAY_APP_SIGNING_SHA256` を **Play アプリ署名の SHA-256** に置換。
  - 取得: Play Console → 対象アプリ → テストとリリース → アプリの整合性 → アプリ署名鍵証明書 の SHA-256。
  - ※ アップロード鍵ではなく **Google が再署名する「アプリ署名鍵」** の SHA-256 を使うこと。複数（アップロード鍵も）入れてもよい。
  - ローカルの鍵から出す場合: `keytool -list -v -keystore <key.jks> -alias <alias>` の SHA256。

## 既に設定済み（アプリ側・このリポジトリで対応済み）
- iOS `Runner.entitlements`: `applinks:outilog.com`
- Android `AndroidManifest.xml`: `<data android:scheme="https" android:host="outilog.com" />`（`autoVerify=true`）
- 共有メッセージのリンク: `https://outilog.com/invite?code=...`
- カスタムスキーム `outilog://invite?code=...` の処理（`deep_link_handler.dart` → 自動参加）

## iOS の appID（参考）
- `CB74J5473A.com.prod.outiLog`（TeamID.BundleID）

## 確認（実機）
1. アプリをインストールした端末で `https://outilog.com/invite?code=XXXX` をメモアプリ等に貼ってタップ → アプリが開いて自動参加。
2. アプリ未インストール端末でタップ → `/invite` ページ → ストアへ誘導。
3. iOS は AASA のキャッシュが数時間残ることがある。反映しない時は再インストールで確認。
