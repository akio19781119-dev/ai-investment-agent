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
