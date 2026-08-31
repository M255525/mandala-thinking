# CLAUDE.md — mandala-thinking

「曼陀羅思考法產生器」——單檔前端工具，把曼陀羅九宮格思考法（Mandala Chart）做成互動網頁：輸入主題，展開成中央 3×3「主題與 8 大延伸面向」（表格 A），每個面向再各自展開一張 3×3「8 個子想法」（表格 B），共 9 張九宮格。核心互動模型是 **manual-first**：所有 72 個格子皆可直接點擊手動編輯，AI 只是加速展開的選用功能，貼近曼陀羅思考法「腦力激盪工作表」的本質，而非純 AI-only 工具。

## 架構

單一 `index.html`：CSS/JS 全內嵌、無外部資源、無建置步驟。視覺主題是深綠「食療/健康」風格（`--bg #0c1a14` + 薄荷綠 `--accent #6ee7b7`），與姊妹專案（青/琥珀/洋紅/紫）區隔。

- **資料結構**（`state`，存 `localStorage` key `mandalaState`）：`{topic, aspects:[8], sub:[8][8], summary, actions:[3]}`。`aspects[i]` 是第 i 個面向名稱，`sub[i][j]` 是第 i 個面向下第 j 個子想法。
- **渲染**：`renderCoreGrid()` 畫表格 A（中心格同步 `topicInput`）；`renderAspectBlocks()`/`renderAspectGrid(i)` 畫 8 張表格 B；`renderJumpNav()` 畫跳轉導覽列。所有 `<textarea>` 的 `input` 事件直接寫回 `state` 對應欄位並 `persistState()`。
- **PRESETS**：5 組內建範例（`行銷內容工具/mandala-thinking/index.html` 內 `PRESETS` 陣列），第一組「食療送餐」是手打（非 AI）示範內容，其餘 4 組跨領域虛構主題（不含真實個資/機構）。
- **BYOK AI**：與 `ai-prompt-generator`／`Prompt` 同一套 `callLLM()`（逐字複製，未修改）——四家服務商（Claude/OpenAI/Gemini/OpenRouter）、逕自瀏覽器直連、逾時 180 秒、429/500/503/529 自動重試。設定存 `localStorage`（key `mandalaApiConfig`）。
  - 「🧠 AI 一鍵展開全部」：`buildFullPrompt()` 要求 AI 以 JSON 回覆 `{aspects[8], sub[8][8], summary, actions[3]}`，`extractJson()` 用正則抓出回應中第一個 `{...}` 區塊再 `JSON.parse`（AI 有時會夾帶額外說明文字，不能直接整段 parse）。
  - 每張表格 B 旁「🧠 AI 重新生成此面向」：`regenerateAspect(i)` 只呼叫 `buildAspectPrompt()`，只覆蓋 `state.sub[i]`，不動其他面向。
- **匯出**：`buildFullText()` 把 9 張表格＋總結＋行動建議組成純文字，供「複製為文字」「下載 .txt」使用；「🖨 列印 / 存 PDF」走 `window.print()` + `@media print` 樣式（隱藏跑馬燈/頂欄/AI面板/按鈕，格子改白底黑字）。
- **列印/PDF 浮水印**：`#printWatermark`（`<img id="wmImg">` 內嵌 base64 品牌圖示「馬克老師 AI・工具・學習・成長」，跟 `IPA_Kano/滿意度分析互動儀表板_進階版.html`／`restaurant-feasibility-calculator/index.html` 同一張已去背 PNG，480×297，用 Python 腳本直接字串複製過來，未經對話視窗），平常 `display:none`，只在 `@media print` 用 `position:fixed`+`display:flex` 顯示、`opacity:.11`、`width:42%`（`min/max-width` 夾住），`position:fixed` 讓每一頁都重複出現。用 `<img>` 而非 CSS `background-image`（後者會被「列印背景圖形」選項預設擋掉）。只套用在列印/PDF 這個輸出管道，不影響「複製為文字」「下載 .txt」。（原本第一版是純文字「僅供參考」，已改成品牌圖示浮水印。）

## 與姊妹專案的差異

- **不加序號授權機制**（`#licenseGate`／`Code.gs`）——本次使用者未要求鎖工具，比照 `coffee-ig-planner`／`social-post-grader` 無授權的先例，序號系統與 BYOK/跑馬燈本就是三個互不相干的獨立系統，可完全略過。
- **manual-first 設計**：其餘姊妹專案的核心互動是「先組成提示詞→選用送給AI」，本工具反過來是「格子本身就是最終產出，AI 只是灌入格子內容的一種方式」——所有格子平常就是 `<textarea>`，沒有「組成」這個中間步驟。

## 共用模組（直接複製既有程式碼）

- **跑馬燈**：`#marqueeBar` 逐字複製 `ai-prompt-generator/index.html` 的 IIFE，共用同一個 Google Apps Script 端點與 Sheet（`localStorage` key 改為 `mandalaMarquee`）。改內容直接編輯共用 Sheet，不需重新部署。
- **創作者資訊／使用警語**：`manual.html`／`index.html` footer 逐字複製既有共用內容（第 8 個同步此內容的專案），第一點警語措辭改成貼合曼陀羅九宮格輸出。
- **PWA**：`manifest.json`／`service-worker.js`／`#installBtn` 邏輯比照既有 9 個已加裝 PWA 的專案（network-first + 同源快取備援，不追求離線可用）。圖示（`icons/`）用 Python PIL 現畫的深綠底＋薄荷綠井字九宮格圖案（非既有素材）。
- **訪客計數器**：`visitor-badge.laobi.icu`，`page_id=m255525.mandala-thinking`。

## Port

**8802**（工作區 8765-8801 已全數被其他專案占用；派工當下 8801 剛被另一個並行 session 的 `business-idea-generator` 佔用，故本專案改用 8802）。

## 指令

無建置/測試指令。修改 `index.html` 後直接用瀏覽器開啟驗證，或暫起 `python -m http.server 8802` 測完關閉。修改內嵌 `<script>` 後可用以下方式快速檢查語法：

```bash
python -c "
import re
html = open('index.html', encoding='utf-8').read()
open('_check.js','w',encoding='utf-8').write('\n\n'.join(re.findall(r'<script>(.*?)</script>', html, re.S)))
"
node --check _check.js
```

**瀏覽器自動化測試踩坑**：這個沙箱環境對外部 Google Apps Script（跑馬燈端點）回應的後續 googleusercontent.com 資源會呈現「pending」狀態，導致 Chrome 分頁的 `document_idle`／screenshot 工具持續回報「頁面忙碌」，但 `document.readyState` 其實已是 `complete`、頁面完全可互動——這不是本工具的程式碼問題（同一段跑馬燈程式碼已在 7+ 個姊妹專案正式環境正常運作），改用 `javascript_tool` 直接查詢 DOM／`localStorage` 驗證功能即可，不需要依賴螢幕截圖。

## 部署

已推公開 GitHub repo `M255525/mandala-thinking`，用 `.github/workflows/deploy-pages.yml`（比照 `social-post-grader` 逐字複製）以 Actions workflow 部署 GitHub Pages（非 legacy branch-source），已上線：<https://m255525.github.io/mandala-thinking/>。

## 本次未做（後續視需要再處理）

- 桌面版 exe 打包
- 序號授權（使用者本次未要求；若之後要鎖工具，比照 `ai-prompt-generator` 的「鎖整個工具 12 個月」模式加回）
