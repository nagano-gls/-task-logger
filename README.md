# Task Logger for Akari v0.7.0

タスク・スケジュール・出退勤を一元化する「コックピット」アプリ。
1ファイルのHTML（`index.html`）で動作し、データはブラウザのlocalStorageに保存されます。

## v0.7.0 の変更点（v0.6.6 → v0.7.0）

| 機能 | 内容 |
|---|---|
| 🔗 リンク集の常時表示 | 右下のフローティングボタン（クリックで開閉）を廃止し、タブ直下に**リンクバーとして常時表示**。設定タブの「リンク集」で登録したリンクがそのまま一覧表示されます |
| 🗓️ Outlook予定表 連携 | 新しい「予定」タブに今日〜2週間の予定を表示。タスクタブ上部にも「今日の予定」を表示。繰り返し予定（毎日/毎週/毎月）・例外・キャンセルにも対応 |
| 🕐 freee勤怠打刻 連携 | 出勤・退勤ボタンを押すと**freeeの打刻ページを自動で新しいタブで開く**（設定でON/OFF）。ヘッダーの「🕐 打刻」ボタンからもいつでも開けます |
| 📱 モバイル/タブレット対応 | 画面幅640px以下ではタブが**下部ナビゲーション**に切り替わるレスポンシブUI。PWA対応（ホーム画面に追加でアプリとして起動、オフラインでも開ける） |

## GitHub Pages で公開する手順

1. GitHubで新しいリポジトリを作成（例: `task-logger`）
   - **注意**: 無料プランでGitHub Pagesを使う場合、リポジトリはPublicにする必要があります。アプリのコード自体にタスクや個人データは含まれません（データは各端末のブラウザ内のみ）が、Outlook ICSはActionsのSecretに登録し、リポジトリに直接書かないでください。
2. このフォルダをpush:
   ```
   git remote add origin https://github.com/<ユーザー名>/task-logger.git
   git push -u origin main
   ```
3. リポジトリの **Settings → Pages → Source: Deploy from a branch → main / (root)** を選択
4. 数分後 `https://<ユーザー名>.github.io/task-logger/` でアクセス可能
5. スマホ/タブレットでそのURLを開き、「ホーム画面に追加」でアプリとして利用

## Outlook予定表 連携のセットアップ

1. [Outlook on the web](https://outlook.office.com/calendar/) を開く
2. **設定（⚙）→ 予定表 → 共有予定表 → 予定表の公開** で予定表を選び、「すべての詳細を閲覧できる」で公開 → **ICSリンク**をコピー
3. 連携方法は2通り（両方設定すると確実）:
   - **アプリに直接登録**: アプリの設定タブ「Outlook予定表 連携」にICSリンクを貼り付け → ブラウザによってはCORS制限で取得できない場合あり
   - **GitHub Actions経由（推奨・確実）**: リポジトリの **Settings → Secrets and variables → Actions → New repository secret** で
     - Name: `OUTLOOK_ICS_URL`
     - Secret: コピーしたICSリンク
     を登録。`.github/workflows/fetch-calendar.yml` が30分ごとにICSを取得して `calendar/outlook.ics` に保存し、アプリが自動でそれを読み込みます。
     初回はリポジトリの **Actions → Fetch Outlook Calendar → Run workflow** で手動実行すると即反映されます。
4. 会社のポリシーで予定表の公開が無効化されている場合は、Outlookから予定表を`.ics`エクスポートして、設定タブの「.icsファイルを読み込む」で手動読み込みも可能です。

## freee勤怠打刻 連携について

freee側のセキュリティ仕様上、外部アプリからの自動打刻はできないため、
「出勤/退勤ボタン押下 → freee打刻ページを自動で開く → freee側で打刻ボタンを押す」という1クリック追加の連携方式です。
打刻ページURLは設定タブで変更できます（既定: `https://kintaiplus.freee.co.jp/independent/recorder2/personal/`）。

## データについての注意

- タスク・記録などのデータは**各端末のブラウザ（localStorage）に保存**されます。PCとスマホの間で自動同期はされません。
  - 端末間でデータを移す場合は、設定タブの「エクスポート」→ 別端末で「インポート」を使ってください。
- 退勤時の自動バックアップ（JSONダウンロード）は従来どおり動作します。

## ファイル構成

```
index.html                          アプリ本体（1ファイル完結）
manifest.webmanifest                PWAマニフェスト
sw.js                               Service Worker（オフラインキャッシュ）
icon-192.png / icon-512.png         アプリアイコン
.github/workflows/fetch-calendar.yml  Outlook ICS定期取得（30分ごと）
calendar/outlook.ics                Actionsが生成する予定データ（自動更新）
```
