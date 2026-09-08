# Portfolio Position Manager

本目錄係俾 Zo 跟住執行嘅持倉管理規格。目標係建立一個半自動 Portfolio Position Manager，用嚟分析全部 HK / US 持倉，而唔係建立自動交易 bot。

## 目標

系統要回答：

- 邊隻持倉到咗 A 點，可以入貨或 DCA。
- 邊隻持倉到咗 B 點，可以對該輪有利潤 lot 止賺 30%。
- Covered call ETF（例如 2802.HK、3416.HK）是否接近封頂，需要止賺提醒。
- 除淨前是否值得用 dividend play 倉做短線收息。
- 每隻持倉最少 30% core income 底倉是否仍然保留。

## 範圍

適用於全部持倉，包括但不限於：

- HK：2802.HK、3416.HK、3110.HK
- US：VT、QQQ、QQQM、SCHD

## 明確不做

- 不自動落盤。
- 不用 kronos-mini-v1 作為 deploy 買賣訊號。
- 不把 `derived_portfolio_snapshot.json` 當 source of truth。
- 不把 profit take 當 bucket。
- 不在未通過 validation checklist 前建立正式資料檔。

## Source of truth

正式資料以原始記錄為準：

1. `position_lots.json`：目前每一輪 lot 的持有狀態。
2. `trade_transactions.json`：所有買入、賣出、止賺交易。
3. `dividend_records.json`：所有派息、收息記錄。
4. `covered_call_events.json`：covered call ETF 的封頂、除淨、派息事件。

`derived_portfolio_snapshot.json` 只係由以上資料計算出來的顯示結果，唔可以手動作真相。

## Bucket 定義

只允許以下 bucket：

- `core_income`：長收息底倉，整體最少保留 30%。
- `dca_growth`：DCA 儲倉及增長倉。
- `dividend_play`：除息短線倉，目標最多 30%。

`profit_take` 係 sell action，不是 bucket。

## 已鎖定策略參數

| 參數 | 值 |
| --- | --- |
| 半自動 | 系統提示，用戶自己落盤 |
| core income 底倉 | 最少 30% |
| dividend play 倉 | 30% |
| B 點止賺 | 該輪有利潤 lot 賣 30% |
| DCA | 每月固定 + 跌到 A 點額外加倉 |

## 執行順序

1. 用 `schema-v2.md` 建立正式 schema。
2. 用 `examples/2802-lot-example.json` 驗算 500 股 2802.HK 範例。
3. 用 `validation-checklist.md` 做建檔前檢查。
4. 檢查通過後，Zo 才可以建立 6 份 JSON 資料檔。
5. 之後才建立每日 / 每週 automation。

## 相關文件

- [schema-v2.md](schema-v2.md)
- [validation-checklist.md](validation-checklist.md)
- [zo-action-prompt.md](zo-action-prompt.md)
- [examples/2802-lot-example.json](examples/2802-lot-example.json)
