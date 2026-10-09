# Zo Action Prompt

將以下內容 copy 到 Zo，要求 Zo 按本 repo 文件執行。

```text
請按 GitHub repo 入面的 Portfolio Position Manager 文件執行，不要自由發明 schema。

文件位置：
- docs/portfolio-position-manager/README.md
- docs/portfolio-position-manager/schema-v2.md
- docs/portfolio-position-manager/validation-checklist.md
- docs/portfolio-position-manager/examples/2802-lot-example.json

目標：
建立一個 Portfolio Position Manager，用於全 portfolio 持倉分析，包括 HK + US 持倉。系統只做半自動提醒，不自動落盤。

核心規則：
1. position_lots.json 和 trade_transactions.json 是 source of truth。
2. derived_portfolio_snapshot.json 只係 derived-only，不可手動作真相。
3. bucket 只可為 core_income / dca_growth / dividend_play。
4. profit_take 只可作 action，不可作 bucket。
5. core_income 底倉最少 30%。
6. dividend_play 倉目標 30%。
7. B 點止賺 = 該輪有利潤 lot 賣 30%。
8. 用戶手動落盤，Zo 只提醒。
9. kronos-mini-v1 不可作 deploy 買賣信號。

第一步：
先根據 schema-v2.md 和 examples/2802-lot-example.json 做 dry-run validation。

請輸出：
1. validation-checklist.md 每一項 PASS / FAIL。
2. 2802 例子的股數、成本、P/L 驗算表。
3. 如全部 PASS，才建議建立 6 份 JSON 檔。
4. 如任何 FAIL，列出要修的地方，不要建立檔案。

禁止：
- 不自動落盤。
- 不改 dashboard。
- 不建立 automation。
- 不用 Kronos 作買賣訊號。
- validation 未通過前，不建立正式 JSON。

輸出語言：繁體中文。
```

## 建立正式檔案時使用

當 dry-run validation 全部通過後，才用以下 prompt：

```text
validation 已全 PASS。

請根據 docs/portfolio-position-manager/schema-v2.md 建立正式資料檔：
1. portfolio_config.json
2. position_lots.json
3. trade_transactions.json
4. dividend_records.json
5. covered_call_events.json
6. derived_portfolio_snapshot.json

要求：
- 初始內容可用 examples/2802-lot-example.json 拆分成 6 份檔案。
- derived_portfolio_snapshot.json 必須保留 derived_only: true。
- 不建立 automation。
- 不改 dashboard。
- 不接 auto trade。

完成後輸出檔案路徑和 validation summary。
```
