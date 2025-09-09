# Morning TimeTree

![workflow](https://github.com/euro0707/Morning-TimeTree/actions/workflows/notify.yml/badge.svg?branch=main)
![license](https://img.shields.io/badge/license-MIT-green)

毎朝 06:00 JST に TimeTree の当日予定を通知します。通知方式は LINE Messaging API を推奨・既定とし、互換のために LINE Notify にも対応します（非推奨）。
重要: LINE Notify はサービス終了済みのため、基本的に LINE Messaging API を利用してください。参考: https://blog.socialplus.jp/knowledge/solution-to-replace-line-notify/

## セットアップ

1) GitHub Secrets を設定（Settings → Secrets and variables → Actions）
- TimeTree 認証:
  - `TIMETREE_EMAIL`: TimeTree ログインメール
  - `TIMETREE_PASSWORD`: TimeTree パスワード
  - `TIMETREE_CALENDAR_CODE`: カレンダーURL末尾の英数字
- 通知方式（推奨: Messaging API）
  - 推奨: `LINE_CHANNEL_ACCESS_TOKEN`, `LINE_USER_ID`（両方あれば自動的に Messaging API を使用）
  - 互換/非推奨: `LINE_TOKEN`（LINE Notify 個人トークン。サービス終了のため基本は使用しない）
- うまくエクスポートできない場合のフォールバック（任意）
  - `TIMETREE_CALENDAR_NUMBER`: カレンダー選択番号（1, 2, ...）。`timetree_exporter` が対話選択を要求する場合に使用します。

2) 内容確認（任意・ローカル）
- Python 3.12+ を用意し、依存をインストール:
  - `pip install -r requirements.txt`
- `.env` を用意すると `python-dotenv` で自動読込されます（コミット禁止。`.env.example` 参照）。
- 整形のみ確認（送信なし）:
  - `python src/notify.py --ics <path/to/your.ics> --date 2025-09-07 --dry-run`
- Messaging API で実送信テスト（ローカル）:
  - `python src/notify.py --ics <path/to/your.ics> --date 2025-09-07 --line-mode messaging --line-channel-access-token "<token>" --line-user-id "<userId>"`

3) GitHub Actions を実行
- 手動: Actions → "Morning TimeTree" → Run workflow（date は任意の `YYYY-MM-DD`）
- 定期: 毎日 06:00 JST（`cron: 0 21 * * *`）に自動通知
- Secrets に `LINE_CHANNEL_ACCESS_TOKEN` と `LINE_USER_ID` が両方あれば Messaging API を優先使用します

## 実装の要点
- タイムゾーン: `zoneinfo` で JST に正規化
- 当日抽出: `[今日00:00, 翌日00:00)`（排他的終端）。終日・跨ぎイベントに対応
- 表示: 終日 → 時間順。場所はあれば `＠場所`。予定なしは `🟢 予定はありません`
- 文字数: 1000 文字上限で切り詰め（Notify基準）。Messaging API でも同じ整形を適用
- 安全性: Secrets のみ使用。ジョブ権限は `contents: read` 最小権限。生成した `calendar.ics` は後片付けで削除
- ローカル: `python-dotenv` により `.env` を自動読込（既存の環境変数は上書きしません）

## トラブルシュート
- `calendar.ics が生成されませんでした`:
  - `timetree-exporter` の CLI 名や引数が環境で異なる場合があります。ワークフローの Export ステップを実環境の `--help` に合わせて調整してください。
- 日本語や絵文字が文字化けする:
  - ローカル実行時は `-X utf8` を付けるか、`PYTHONUTF8=1` を設定してください。
- 通知が来ない:
  - Actions 実行ログで各ステップ（Export/Send）を確認。Secrets とスケジュールの有効性、Bot の友だち追加/プッシュ有効化を点検

## 参考
- セキュリティ指針: `SECURITY.md`

## ライセンス
- 本リポジトリはオープンソース（MIT License）です。詳細は `LICENSE` を参照してください。
