# Crypto Volume Scanner

Binance USDT 市場成交量異常掃描器，包含即時 Dashboard、異常觀察清單、量化樣本紀錄，以及每日多空建議 agent。

此專案只提供量化觀察與研究用途，不是投資建議。實際交易請自行控管倉位、停損與風險。

## 功能

- 掃描 Binance Spot / USD-M Perp USDT 交易對。
- 以 rolling 5m quote volume 計算成交量倍率與 z-score。
- 追蹤 taker buy / sell ratio，判斷買盤或賣盤是否主導。
- Dashboard 即時顯示推薦標的、一般觀察清單、異常監控室。
- 多空濾網整合 EMA、RSI、MACD、order book depth、recent trade flow。
- SQLite 紀錄異常訊號與量化樣本。
- `daily_market_agent.py` 可產出每日多空建議。
- `watch` 模式可長時間累積樣本，讓規則越跑越有統計依據。

## 安裝

```powershell
pip install -r requirements.txt
Copy-Item .env.example .env
```

## 啟動 Dashboard

```powershell
python -m uvicorn app:app --host 127.0.0.1 --port 8000
```

打開：

```text
http://127.0.0.1:8000
```

## 啟動長時間紀錄 Agent

```powershell
powershell.exe -ExecutionPolicy Bypass -File .\run_watch_market_agent.ps1
```

預設行為：

- 每 5 分鐘抓一輪市場樣本。
- 補齊 5m / 15m / 30m 到期績效。
- 每小時刷新 `reports/daily_market_advice.md`。
- 持續寫入 `signals.db`。

## 每日多空建議

手動執行：

```powershell
python -B daily_market_agent.py daily
```

只產生建議、不新增樣本：

```powershell
python -B daily_market_agent.py advice
```

## 量化紀錄

手動抓多輪樣本：

```powershell
python -B quant_volume_recorder.py record --rounds 6 --interval 10
```

補到期績效：

```powershell
python -B quant_volume_recorder.py update-outcomes --allow-market-fallback
```

輸出量化報告：

```powershell
python -B quant_volume_recorder.py analyze
```

## 重要檔案

- `app.py`：FastAPI Dashboard。
- `main.py`：WebSocket 掃描主流程。
- `realtime_store.py`：即時行情記憶體資料層。
- `db.py`：SQLite 儲存層。
- `detector.py`：成交量異常與方向判斷。
- `quant_volume_recorder.py`：量化樣本紀錄器。
- `daily_market_agent.py`：每日多空建議 agent。
- `backup_data.py`：搬家用資料庫備份工具。
- `MARKET_AGENT.md`：agent 使用說明。
- `MIGRATION.md`：搬到另一台電腦的流程。

## 搬移到新電腦

請看 [MIGRATION.md](MIGRATION.md)。

簡短版：

1. GitHub 只放程式碼。
2. `.env`、`signals.db`、log 不要上傳。
3. 新電腦 clone 後，安裝 requirements，複製 `.env.example` 成 `.env`。
4. 若要保留歷史量化資料，另外手動搬 `signals.db` 相關檔案。
