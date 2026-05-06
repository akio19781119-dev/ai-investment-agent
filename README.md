# ai-investment-agent

## Colab実行手順（初心者向け）
1. `us_stock_ai_report_mvp_colab.ipynb` をColabで開く。
2. **セル1 → セル2 → セル3** の順で実行（`Runtime > Run all` でも可）。
3. レイアウト確認だけしたい場合はセル2で `dry-run = y` を選択。

## 今回の改善ポイント
- PDFを **A4横向き** で生成（余白小さめ、見やすいカード風構成）。
- 1ページ目に「本日の結論」、2ページ目に「サマリー」と「指標定義」を追加。
- YouTube取得の頑健性を改善（`yt-dlp` 更新、Node.js導入、失敗時フォールバック）。
- YouTube失敗時でもRSS / yfinanceでPDF生成を継続。
- `dry-run` 時はメール送信のみスキップし、PDFとログは保存。

## 設定（report / design）
ノートブック内部の `CONFIG` に以下の拡張設定を追加済みです。
- `report.language`, `report.title`
- `report.page_size`, `report.orientation`
- `report.include_cover_image`, `report.cover_image_path`
- `report.include_summary`, `report.include_metric_definitions`
- `report.top_candidates_count`, `report.avoid_candidates_count`
- `report.include_disclaimer`, `report.include_source_urls`
- `design.theme`, `design.primary_color`, `design.accent_color`, `design.background_color`, `design.card_background`

## assetsフォルダと表紙画像
- assetsフォルダ: `/content/drive/MyDrive/ai-investment-agent/assets`
- 置き場所: `/content/drive/MyDrive/ai-investment-agent/assets/cover.png`
- `cover_image_path` は「表紙に使う画像ファイルの場所」です。
- 画像がない場合は、処理を止めずに代替表紙（明るい背景）でPDF生成を継続します。

## 指標の意味
- **Upside Score**: 今後2年の上昇可能性を100点で示す参考指標。
- **Risk Score**: 変動性・割高感・不確実性などの注意指標。
- **Momentum**: 直近の株価の勢い。
- **Valuation**: 利益や成長性に対して割高/割安かを見る考え方。
- **Catalyst**: 決算・新製品・金利変化など株価を動かす材料。
- **Source Confidence**: 情報源の信頼度。
- **Cramer Dependence**: Cramer発言への依存度。
- **Macro Fit**: 金利・インフレ・景気などマクロ環境との適合。

## dry-run確認方法
- セル2で `dry-run = y` にする。
- セル3実行後、以下を確認:
  - A4横向きPDFが `reports` に保存される
  - 表紙（画像または代替）
  - 「本日の結論」「サマリー」「指標定義」表示
  - コンソールに「dry-runのためメール送信はスキップしました。」

## メール送信とエラー時の動作
- 本番実行（dry-runでない）かつ `SEND_EMAIL=True` の場合のみ Gmail SMTP 送信。
- メール送信に失敗しても、PDF保存とログ保存（`logs`）は残る設計です。

## Colab最新版レポート運用（Jim Cramer / Ray Dalio / Scott Galloway）
- 指定ソース: DeepCramerJP（Cramer解説）, CNBC Television（Cramer/Mad Money直接性高）, Makabee（Cramer解説）, Principles by Ray Dalio（Dalio公式）, Pivot（Scott Galloway関連Podcast）。
- dry-run: 取得・debug保存・品質チェックまで実行し、品質通過時のみPDF生成。メール送信はしません。
- 本番実行: 取得・debug保存・品質チェック後、品質通過かつPDFサイズ/認証情報条件を満たした場合のみGmail送信します。
- debug保存先: `/content/drive/MyDrive/ai-investment-agent/debug/`
  - `raw_sources_YYYY-MM-DD.json`: 取得生データ
  - `source_check_YYYY-MM-DD.csv`: ソース別取得件数/失敗原因
  - `extracted_mentions_YYYY-MM-DD.csv`: 抽出銘柄/会社名
  - `mentioned_by_person_YYYY-MM-DD.csv`: 3者横並び比較用データ
  - `report_quality_check_YYYY-MM-DD.json`: 品質ゲート結果
- `source_check.csv` は `fetched_items_count`, `status`, `error_message` を最初に確認してください。
- `extracted_mentions.csv` は `validated` が false でも削除せず「要確認」として扱います。
- `mentioned_by_person.csv` で Cramer / Dalio / Galloway それぞれの stance/reason/source_url を追跡できます。
- 品質チェック失敗時は PDF本番生成/メール送信を停止し、失敗理由を `report_quality_check` とログに出力します。
- Gmail送信されない場合は `EMAIL_TO`, `SMTP_USER`, `SMTP_PASSWORD`, PDFサイズ(50KB以上), `Quality gate passed` を確認してください。
- GitHub版ノートブックをColabで再読込する場合は、GitHub URL指定で開き、`Runtime > Restart and run all` を実行してください。

## 字幕・文字起こし優先の発言分析（新方針）
- 本アプリは**タイトル/説明文中心ではなく、字幕・文字起こし本文中心**で発言分析します。
- 優先順位は「手動字幕 → 自動字幕 →（設定ON時のみ）音声文字起こし → metadata_only」です。
- `metadata_only` は本文根拠が弱いため、confidenceは低く扱います。

## debug CSVの見方
- `statement_units_YYYY-MM-DD.csv`:
  - 発言単位の主データ。speaker/directness/stance/reason/risk/evidenceを保持。
- `source_check_YYYY-MM-DD.csv`:
  - ソース別の取得件数、失敗状態、エラーメッセージ確認用。
- `extracted_mentions_YYYY-MM-DD.csv`:
  - 抽出された銘柄・会社・評価の一覧。
- `mentioned_by_person_YYYY-MM-DD.csv`:
  - Cramer/Dalio/Pivotの3者横並び比較の基データ。

## dry-run と本番実行
- dry-run: PDF生成・debug保存まで実施、Gmail送信はしません。
- 本番: 品質チェック通過 + PDF生成成功時のみGmail送信します（PDF添付のみ）。

## 品質チェック失敗時
- 本番PDF送信を停止し、失敗理由を `report_quality_check_YYYY-MM-DD.json` とログに残します。
- 「品質チェック失敗」と画面・ログへ出力します。

## YouTube WARNING の扱い
- `WARNING: [youtube:tab] Incomplete data received...` は部分失敗として扱い、`ignoreerrors` で継続します。
- 失敗ソースは `source_check` の `status=failed` と `error_message` を確認してください。

## Gmail送信されない場合の確認
1. `DRY_RUN` が `False` か
2. `SEND_EMAIL` が `True` か
3. `EMAIL_TO`, `SMTP_USER`, `SMTP_PASSWORD` が設定済みか
4. `Quality gate passed: yes` か
5. PDFが生成済みか（ログの `PDF generated` / `PDF path`）
