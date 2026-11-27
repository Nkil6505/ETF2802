# Portfolio Position Manager Validation Checklist

Zo 建立正式 JSON 檔案前，必須逐項通過本 checklist。

## 1. Source of truth

- [ ] `position_lots.json` 存目前每一輪 lot 的持倉狀態。
- [ ] `trade_transactions.json` 存所有買入、賣出、止賺交易。
- [ ] `dividend_records.json` 存所有收息記錄。
- [ ] `covered_call_events.json` 存所有 covered call ETF 事件。
- [ ] `derived_portfolio_snapshot.json` 有 `derived_only: true`。
- [ ] 不可由人手直接改 `derived_portfolio_snapshot.json` 當真相。

## 2. Bucket 規則

- [ ] Bucket 只可為 `core_income`、`dca_growth`、`dividend_play`。
- [ ] `profit_take` 只可作 `action`，不可作 bucket。
- [ ] `core_income` 最少 30%。
- [ ] `dividend_play` 目標 30%。
- [ ] `dividend_play` 不可與 `core_income` 底倉重疊。

## 3. 2802 範例股數

以 [examples/2802-lot-example.json](examples/2802-lot-example.json) 為準：

- [ ] Original quantity = 500。
- [ ] `core_income` = 150 = 30%。
- [ ] `dca_growth` = 200 = 40%。
- [ ] `dividend_play` = 150 = 30%。
- [ ] B 點止賺賣出 = 45 = 30% of dividend_play lot。
- [ ] Remaining quantity = 455。
- [ ] Sold quantity = 45。

## 4. Cost basis and P/L

- [ ] 金額計算使用 decimal arithmetic。
- [ ] 展示金額以 half-up 四捨五入到小數點後 2 位。
- [ ] `net_cost_per_share = net_entry_amount / original_quantity`。
- [ ] Sell transaction 必須只 link 真正賣出的 lot。
- [ ] 不可列出 `sell_quantity = 0` 的 lot。
- [ ] `sale_proceeds = sell_price * sell_quantity`。
- [ ] `cost_basis = net_cost_per_share * sell_quantity`。
- [ ] `gross_realized_pnl = sale_proceeds - cost_basis`。
- [ ] `net_realized_pnl = gross_realized_pnl - fees_allocated`。

### 2802 範例核數

- [ ] sale proceeds = 45 * 7.30 = 328.50。
- [ ] cost basis = 45 * 7.1530 = 321.89。
- [ ] gross realized P/L = 328.50 - 321.89 = 6.62。
- [ ] net realized P/L = 6.62 - 0.27 = 6.35。

## 5. Dividend records

- [ ] 每筆 dividend 都有 `ex_date`、`pay_date`、`amount_per_share`。
- [ ] `eligible_quantity` 等於 linked lots eligible quantity 的總和。
- [ ] 每筆 dividend 要有 `linked_lots[]`。
- [ ] `gross_dividend = amount_per_share * eligible_quantity`。
- [ ] `net_dividend = gross_dividend - tax_withheld`。

## 6. Covered call events

- [ ] 每筆事件有 `underlying`，例如 `HSCEI`。
- [ ] 每筆事件有 `strike_or_cap_level`。
- [ ] 每筆事件有 `distance_to_cap_pct`。
- [ ] `cap_status` 只可為 `green`、`yellow`、`red`。
- [ ] `red` 狀態可以觸發 `consider_profit_take` alert。

## 7. Derived snapshot

- [ ] `original_quantity = remaining_quantity + sold_quantity`。
- [ ] `market_value = remaining_quantity * current_price`。
- [ ] `total_cost_basis_remaining` 用 remaining lots 計算。
- [ ] `unrealized_pnl = market_value - total_cost_basis_remaining`。
- [ ] `realized_pnl` 來自 sell transactions。
- [ ] `net_dividends_received` 來自 dividend records。

## 8. Manual update boundaries

用戶只應手動填：

- [ ] 買入：ticker、account、broker、currency、trade date、quantity、price、fees、bucket、tag。
- [ ] 賣出：ticker、account、broker、currency、trade date、quantity、price、fees、sell rule 或 linked lot。
- [ ] 收息：ticker、ex_date、pay_date、amount_per_share、eligible quantity、tax。

以下欄位應由系統計算：

- [ ] remaining quantity。
- [ ] sold quantity。
- [ ] net cost per share。
- [ ] P/L。
- [ ] derived snapshot。

## 9. Approval gate

所有項目通過前：

- [ ] 不建立正式 JSON 檔案。
- [ ] 不建立 automation。
- [ ] 不改 dashboard。
- [ ] 不接入 auto trade。
