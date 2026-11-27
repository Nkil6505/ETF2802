# Portfolio Position Manager Schema v2

本文定義 6 份 JSON 檔案。建立正式檔案前，必須先通過 [validation-checklist.md](validation-checklist.md)。

## 共同規則

- 全部日期使用 ISO 格式。
- 金額欄位使用交易原幣，並以 `currency` 標示。
- 金額計算使用 decimal arithmetic；展示金額以 half-up 四捨五入到小數點後 2 位。
- 不使用 `_hkd` / `_usd` 作通用欄位名稱；需要折算 CNY 時使用 `fx_rate_to_cny`。
- 用戶手動輸入原始事件；系統計算 `remaining_quantity`、`sold_quantity`、P/L、snapshot。
- Bucket 只可為 `core_income`、`dca_growth`、`dividend_play`。
- `profit_take` 只可作 `action`，不可作 bucket。

## 1. portfolio_config.json

用途：保存固定策略設定，不保存現金、總資產、市值等動態數字。

```json
{
  "_schema": "Portfolio configuration",
  "portfolio_id": "portfolio_v2",
  "last_updated": "2026-07-05T15:00:00+08:00",
  "strategy_defaults": {
    "min_core_income_pct": 0.30,
    "dividend_play_pct": 0.30,
    "profit_take_pct_of_profitable_lot": 0.30,
    "use_dynamic_stop_loss": true,
    "dynamic_stop_loss_levels": {
      "core_income": -0.035,
      "dca_growth": -0.050,
      "dividend_play": -0.050
    }
  },
  "risk_settings": {
    "max_drawdown_per_position": -0.05,
    "max_drawdown_portfolio": -0.10
  }
}
```

## 2. position_lots.json

用途：保存目前每一輪 lot 的持有狀態。這是持倉 lot source of truth。

```json
{
  "_schema": "Position lots source of truth",
  "last_updated": "2026-07-05T15:05:00+08:00",
  "lots": [
    {
      "lot_id": "LOT-20260608-001",
      "ticker": "2802.HK",
      "bucket": "core_income",
      "account": "uSMART",
      "broker": "uSMART",
      "currency": "HKD",
      "original_quantity": 150,
      "remaining_quantity": 150,
      "sold_quantity": 0,
      "status": "open",
      "entry_date": "2026-06-08",
      "entry_price": 7.10,
      "gross_entry_amount": 1065.00,
      "fees": 0.50,
      "net_entry_amount": 1065.50,
      "net_cost_per_share": 7.1033,
      "fx_rate_to_cny_at_entry": 0.92,
      "tag": "A點入貨",
      "notes": "Core income lot",
      "created_at": "2026-06-08T09:00:00+08:00",
      "updated_at": "2026-06-08T09:00:00+08:00"
    }
  ]
}
```

### Lot 狀態

- `open`：未賣出。
- `partial`：部分賣出。
- `closed`：全部賣出。

## 3. trade_transactions.json

用途：保存所有買入、賣出、止賺交易原始記錄。

### Buy transaction

```json
{
  "tx_id": "TX-20260608-001",
  "timestamp": "2026-06-08T09:00:00+08:00",
  "ticker": "2802.HK",
  "account": "uSMART",
  "broker": "uSMART",
  "currency": "HKD",
  "action": "buy",
  "lot_id": "LOT-20260608-001",
  "price": 7.10,
  "quantity": 150,
  "gross_amount": 1065.00,
  "fees": 0.50,
  "net_amount": 1065.50,
  "tag": "A點入貨",
  "notes": "Initial core income lot"
}
```

### Sell / profit take transaction

Sell 必須逐 lot link，只列真正賣出的 lots。不可列 `sell_quantity = 0` 的 lot。

```json
{
  "tx_id": "TX-20260705-004",
  "timestamp": "2026-07-05T10:00:00+08:00",
  "ticker": "2802.HK",
  "account": "uSMART",
  "broker": "uSMART",
  "currency": "HKD",
  "action": "profit_take",
  "price": 7.30,
  "quantity": 45,
  "gross_amount": 328.50,
  "fees": 0.27,
  "net_amount": 328.23,
  "linked_lots": [
    {
      "lot_id": "LOT-20260622-003",
      "sell_quantity": 45,
      "net_cost_per_share": 7.1530,
      "sell_price": 7.30,
      "sale_proceeds": 328.50,
      "cost_basis": 321.89,
      "gross_realized_pnl": 6.62,
      "fees_allocated": 0.27,
      "net_realized_pnl": 6.35
    }
  ],
  "source_bucket": "dividend_play",
  "tag": "B點止賺",
  "notes": "Sold 30% of profitable dividend_play lot"
}
```

## 4. dividend_records.json

用途：保存所有派息、收息記錄，並 link 到合資格 lots。

```json
{
  "_schema": "Dividend records source of truth",
  "last_updated": "2026-07-05T15:25:00+08:00",
  "dividends": [
    {
      "dividend_id": "DIV-202606-001",
      "ticker": "2802.HK",
      "month": "2026-06",
      "ex_date": "2026-06-30",
      "pay_date": "2026-07-07",
      "currency": "HKD",
      "amount_per_share": 0.15,
      "eligible_quantity": 255,
      "gross_dividend": 38.25,
      "tax_withheld": 0.00,
      "net_dividend": 38.25,
      "linked_lots": [
        {
          "lot_id": "LOT-20260608-001",
          "bucket": "core_income",
          "eligible_quantity": 150
        },
        {
          "lot_id": "LOT-20260622-003",
          "bucket": "dividend_play",
          "eligible_quantity": 105
        }
      ],
      "notes": "Eligible quantities reflect holdings on ex-date"
    }
  ]
}
```

## 5. covered_call_events.json

用途：保存 covered call ETF 的封頂、除淨、派息、cap risk 事件。

```json
{
  "_schema": "Covered call ETF events",
  "last_updated": "2026-07-05T15:30:00+08:00",
  "events": [
    {
      "event_id": "CC-202607-001",
      "ticker": "2802.HK",
      "underlying": "HSCEI",
      "month": "2026-07",
      "hscei_level": 7699.76,
      "strike_or_cap_level": 7700.00,
      "distance_to_cap_pct": -0.0031,
      "cap_status": "red",
      "alert_action": "consider_profit_take",
      "ex_date": "2026-07-30",
      "pay_date": null,
      "distribution_per_unit": 0.15,
      "notes": "Underlying is slightly below cap; upside is nearly capped"
    }
  ]
}
```

### cap_status enum

- `green`：距離 cap 仍遠。
- `yellow`：接近 cap，需要觀察。
- `red`：貼近或高於 cap，可考慮止賺。

## 6. derived_portfolio_snapshot.json

用途：由 lots、transactions、dividends、covered call events 計算出的 snapshot。不可手動作 source of truth。

```json
{
  "_schema": "Derived portfolio snapshot",
  "derived_only": true,
  "source_files": [
    "portfolio_config.json",
    "position_lots.json",
    "trade_transactions.json",
    "dividend_records.json",
    "covered_call_events.json"
  ],
  "last_updated": "2026-07-05T15:35:00+08:00",
  "positions": [
    {
      "ticker": "2802.HK",
      "account": "uSMART",
      "currency": "HKD",
      "original_quantity": 500,
      "remaining_quantity": 455,
      "sold_quantity": 45,
      "current_price": 7.12,
      "market_value": 3239.60,
      "total_cost_basis_remaining": 3241.67,
      "unrealized_pnl": -2.07,
      "realized_pnl": 6.35,
      "net_dividends_received": 38.25,
      "core_income_qty": 150,
      "dca_growth_qty": 200,
      "dividend_play_original_qty": 150,
      "dividend_play_sold_qty": 45,
      "dividend_play_remaining_qty": 105,
      "core_income_pct_of_original": 0.30,
      "dividend_play_pct_of_original": 0.30,
      "latest_cap_status": "red",
      "next_alerts": [
        "CAP_RED_CONSIDER_PROFIT_TAKE",
        "DIVIDEND_PLAY_REVIEW_AFTER_PAY_DATE"
      ]
    }
  ]
}
```

## Manual update fields

用戶手動只應填原始事件資料。

### 買入

- ticker
- account
- broker
- currency
- trade date
- quantity
- price
- fees
- bucket
- tag / notes

### 賣出

- ticker
- account
- broker
- currency
- trade date
- quantity
- price
- fees
- sell rule 或 linked lot

### 收息

- ticker
- ex_date
- pay_date
- amount_per_share
- eligible_quantity
- tax_withheld

其餘 remaining quantity、sold quantity、net cost、P/L、snapshot 由系統計算。
