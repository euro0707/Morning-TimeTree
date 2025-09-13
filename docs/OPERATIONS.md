# Morning TimeTree 運用メモ

このプロジェクトを安全・簡潔に運用するための要点まとめです。初回セットアップ後の保守に役立ててください。

## スケジュール（cron と時刻）
- GitHub の `cron` は UTC で評価されます。
- 毎日 06:00 JST は `0 21 * * *`（前日 21:00 UTC）。
- 平日のみ: `0 21 * * 1-5`。
- 複数時刻は `- cron:` 行を複数追加。
- 一時停止: Actions 画面で対象 Workflow を Disable、または `on.schedule` をコメントアウト。

## Secrets（必須と保管場所）
- Repository Secrets に保存（Settings → Secrets and variables → Actions）。
- TimeTree: `TIMETREE_EMAIL`, `TIMETREE_PASSWORD`, `TIMETREE_CALENDAR_CODE`（必要に応じて `TIMETREE_CALENDAR_NUMBER`）。
- LINE 通知（推奨）: `LINE_CHANNEL_ACCESS_TOKEN`, `LINE_USER_ID`。
- LINE Notify（非推奨・互換）: `LINE_TOKEN`。
- ローテーション時は旧トークンを revoke（失効）し、再実行で到達確認。

## Workflow の前提と設計
- `environment:` は未使用（承認待ちを避けるため）。
- `permissions: contents: read`（最小権限）。
- `concurrency: morning-timetree`（同時実行防止）。
- Python 3.12。依存は `requirements.txt`。`timetree-exporter` は挙動差があるため未ピン留め（安定後に固定検討）。

## TimeTree エクスポートの注意
- `timetree-exporter` は配布形態により CLI 名や引数が異なることがあるため、
  - まず CLI 実行（`timetree-exporter`）
  - 失敗時はモジュール実行（`python -m timetree_exporter`）にフォールバック。
- 生成に失敗した場合は明示的にジョブを失敗させます（`calendar.ics が生成されませんでした`）。

## 通知メッセージ仕様
- 対象期間: 当日 00:00 〜 翌日 00:00（終端は排他的）。終日・日跨ぎに対応。
- 並び順: 終日 → 時間順。場所は `＠場所` で付与。
- 文字数: 上限 1000 文字。溢れた分は `…ほかN件` を付与。

## 権限とブランチ保護
- Ruleset `main-protection` を使用。
- Repository admin を Bypass（PR 要件と Push 制限を迂回可）。
- 直接 push が拒否されたら Rulesets の Bypass 設定（または組織側 Ruleset）を確認。

## トラブルシュート
- 通知が来ない: Actions ログで `Export TimeTree ICS` / `Send LINE notification` を確認。Secrets、時刻、Bot の友だち追加・プッシュ許可を点検。
- `Waiting for review: production…`: Environments → `production` → Required reviewers を OFF（この Workflow では `environment:` 未使用）。
- 文字化け: ローカル実行時は `PYTHONUTF8=1` か `-X utf8` を使用。

## ローカル確認（任意）
```bash
pip install -r requirements.txt
# 整形のみ（送信なし）
python src/notify.py --ics calendar.ics --date 2025-09-07 --dry-run
# Messaging API で送信
python src/notify.py --ics calendar.ics --date 2025-09-07 \
  --line-mode messaging \
  --line-channel-access-token "<token>" --line-user-id "<userId>"
```

