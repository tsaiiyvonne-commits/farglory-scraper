# 商仲報告爬蟲專案 (Real Estate Scraper)

這是一個自動化的 Python 網頁爬蟲與系統拋轉框架。目的是針對指定的全球商仲與顧問機構（包含 JLL, Colliers, CBRE 等 10+ 家機構）定期抓取其最新的市場研究報告（PDF），並串接 TargetSystem 平台 API，達成地產市場資訊收集的自動化。

本專案提供已經過驗證的架構底座、反爬蟲繞過技術示範，以及與 TargetSystem 介接的相容標準。

## 🎯 專案目標與任務

1. **定時排程抓取**：設定 Cronjob 或排程腳本，定期拜訪各家商仲的研究報告頁面。
2. **反爬蟲突破**：內建 `curl_cffi` 技術，能成功模擬 Chrome 120 的 TLS 指紋，突破 Colliers 或澳洲央行 (RBA) 等使用的 Cloudflare WAF 防火牆。
3. **實體檔案下載**：將抓取到的公開 PDF 報告，自動儲存至 iCloud 的共享資料夾內。
4. **TargetSystem API 整合**：採用 Compatible API (JSON Format) 標準，將最新檔案的儲存路徑與 Metadata（如報告來源、報表時間等）推送給 TargetSystem 應用程式。

## 📂 專案檔案結構

* `scraper_template.py`：主執行檔。包含了 Config 設定、四大爬蟲模組（爬蟲邏輯、下載器、平台對接、排程主函數）。
* `requirements.txt`：Python 依賴套件清單。
* （產出物） `All_Reports_Scraping_Analysis.xlsx`：提供給開發工程師的終極目標網址與爬取難度評估表（共 55 份澳洲與倫敦報告）。

## 🚀 模組化架構開發指南 (給接手的工程師)

本程式採用物件導向（OOP）開發，分為明確的五大模塊。接手工程師只需專注於**「模組 2：擴充各站點專用爬蟲邏輯 (Scrapers)」**即可。

### 1. 安裝環境與相依性

請使用 Python 3.9+ 的虛擬環境：
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

核心套件說明：
* `requests` / `beautifulsoup4`：常規爬蟲解析。
* `curl_cffi`：**[極重要]** 用於繞過 Cloudflare 與強烈防護網站（已內建於模板之 Session）。
* `schedule`：輕量級 Python 排程套件。

### 2. 接手開發 (需實作之 TODO清單)

請打開 `scraper_template.py` 並搜尋 `TODO` 標籤：

1. **設定環境變數 (模組 1)**：更改 `DOBBY_API_ENDPOINT` 以及您的安全 `DOBBY_API_TOKEN`，並確認 `BASE_DOWNLOAD_DIR` 目錄於運行主機上存在有寫入權限。
2. **擴充解析邏輯 (模組 2)**：
   * 目前 `ReportScrapers` 類別內提供 `fetch_jll_reports()` 等空殼方法。
   * 請對照 `All_Reports_Scraping_Analysis.xlsx` 內的網站，實作各家網站的「尋找清單 -> 解析分頁 -> 找到 PDF 連結」邏輯並回傳結構化的 dict list。
   * 遇到阻擋時，請使用類別內的 `self.session.get(..., impersonate="chrome120")`。
3. **實裝下載與發送邏輯 (模組 3 & 4)**：將程式註解中實際發送 Requests 與寫入檔案的段落 (# 取消註解) 啟用。

## ⚠️ 開發注意事項與已知難點

1. **JLL / 部分商仲的 Lead Gen 表單**：這類網站的 PDF 下載連結往往藏在一組名單蒐集表單（填寫 Name / Email / Company）之後。工程師需撰寫額外的自動 POST 表單邏輯。
2. **MSCI (RCA) 付費牆**：此報告來源可能需要實作自動登入機制（帶 Cookie 或 Token Session）才能下載。
3. **客製化報告 (Prepared For ClientName)**：若發現不在外網公開的顧問交付報告，不應使用本系統去網站硬爬。這類目標建議改為編寫 E-mail (如 Outlook/Gmail 附加檔案自動下載) 腳本。

## 執行與部署

本機測試：
```bash
python3 scraper_template.py
```

伺服器上線建議：
* 可使用 Linux 內建的 `Cron` 取代程式內的 `schedule`，或
* 搭配 `Supervisor` 或 `Docker` 來確保 Python 腳本持續運行於背景不中斷。
