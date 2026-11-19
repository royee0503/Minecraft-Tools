# Minecraft Tools

[繁體中文](README.md) | [English](README_EN.md)

---

### 功能

本專案包含兩支獨立的小工具：

- **模組下載器**：依照 `mods.json` 中定義的模組清單，從 Modrinth 下載指定 Minecraft 版本的模組。
- **材質包下載器**：從 Vanilla Tweaks 產生並下載指定 Minecraft 版本的材質包，材質包組合與 TaurusTweaks 相同，並依版本自動選用對應的清單（26.x 以後、1.21 及更舊）。

### 環境需求與執行方式

- Python 3.9 以上
- 使用 Python 內建的 `venv` 建立獨立虛擬環境。

### 建立虛擬環境（Windows PowerShell）

#### 1. 進入專案資料夾
```powershell
cd .\Minecraft-Tools
```

#### 2. 建立獨立虛擬環境
```powershell
python -m venv .venv
```

這會在專案內建立 `.venv` 資料夾：
```text
Minecraft-Tools/
├── .venv/
├── modrinth_downloader.py
├── vanillatweaks_downloader.py
└── ...
```

#### 3. 啟用虛擬環境
```powershell
.\.venv\Scripts\Activate.ps1
```

成功後，PowerShell 提示字元前方會出現 `(.venv)`，例如：
```text
(.venv) PS C:\Users\<使用者名稱>\Minecraft-Tools>
```

#### 4. 確認 Python 使用的是虛擬環境
```powershell
python --version
where.exe python
```

`where.exe python` 顯示的第一個 Python 路徑應位於專案的 `.venv\Scripts\` 資料夾內。

#### 5. 安裝專案依賴
```powershell
python -m pip install requests
```

#### 6. 執行程式
```powershell
python modrinth_downloader.py
python vanillatweaks_downloader.py
```

離開虛擬環境可輸入 `deactivate`。下次重新開啟終端機時，進入專案資料夾並再次執行啟用指令即可。
執行後依照畫面提示輸入 Minecraft 版本即可。兩支程式都可以隨時取消：模組下載器輸入 `q` 或按 Ctrl+C；材質包下載器按 Q（Windows 終端機會立刻取消，其他環境請輸入 `q` 再按 Enter）。

### 安全原則

- 兩支程式都只會新增檔案，絕不覆蓋或刪除你現有的檔案（只會清除自己產生的 `.part` 暫存檔）。
- 模組下載器：下載後會驗證檔案大小與雜湊值（hash，優先用 SHA-512，沒有時用 SHA-1）。沒有雜湊值的檔案不會下載，驗證失敗的檔案不會保留。同名檔案已存在時會直接略過。模組出新版時，若舊版 jar 仍在資料夾中，只會提醒你手動移除，不會自動刪除。
- 材質包下載器：先下載成暫存檔，確認是有效的 ZIP 才存成正式檔案；檔名重複時會自動加上 (1)、(2)。下載完成後會讀取 ZIP 內的 `Selected Packs.txt`，檢查送出的材質包是否都有打包進去。

### 程式結構（由上往下閱讀）

#### `modrinth_downloader.py`

1. 設定與常數
2. 資料結構（`Config` / `ModResult`）
3. 載入設定檔
4. Modrinth API
5. 選擇版本與檔案
6. 下載與驗證
7. 已安裝模組清單（`installed.json`）
8. 處理單一模組
9. 報告
10. 主程式

#### `vanillatweaks_downloader.py`

1. 基本設定（含兩份材質包清單）
2. 自訂錯誤類別
3. 輸入並檢查版本
4. 取得分類資料
5. 請網站產生 ZIP
6. 下載 ZIP
7. 檢查 ZIP 內容
8. 主程式
9. 程式進入點

### 檔案說明

#### `modrinth_downloader.py` - Minecraft 模組下載器（Modrinth）

自動模組下載器的主程式，不需要修改程式本身，只要編輯 `mods.json`。

- 執行時會詢問目標 Minecraft 版本，並用 Modrinth 的版本清單檢查輸入是否正確。
- 依照 `mods.json` 的 Loader 與版本類型，為每個模組挑選最新的合適版本。找不到時會說明原因（例如模組尚未更新、沒有該 Loader 的版本、只有 beta 版），這不算錯誤，之後可以再執行一次。
- 檔案下載到 `<repository_dir>/<Minecraft 版本>/`。
- `installed.json` 記錄每個模組目前下載的檔名，用來偵測舊版 jar；`download_report.json` 是這次執行的完整報告。
- 模組若有必要依賴，只會提醒你另行確認，不會自動下載。

#### `mods.json`

你可以編輯這個檔案，指定要自動下載哪些模組。範例：

```json
{
  "repository_dir": "~/Downloads/Minecraft Mod Repository",
  "loaders": ["fabric"],
  "version_types": ["release"],
  "mods": [
    { "name": "Apple Skin", "project": "appleskin" },
    { "name": "Architectury", "project": "architectury-api" }
  ]
}
```

模組的必要依賴不會自動下載，所以像 Fabric API、Architectury、Cloth Config 這類前置模組，也要一起列進清單（如上面的範例）。

- `mods`（必填）：模組清單。`project` 填 Modrinth 的專案代稱（slug）或專案 ID；`name` 是選填的顯示名稱。
- `loaders`（選填，預設 `["fabric"]`）：要下載哪些 Loader 的版本。
- `version_types`（選填，預設 `["release"]`）：接受的版本類型，可填 `release`、`beta`、`alpha`。
- `repository_dir`（選填，預設 `~/Documents/Minecraft Mod Repository`）：下載位置，底下會依 Minecraft 版本建立資料夾。

#### `vanillatweaks_downloader.py` - Minecraft 材質包下載器（Vanilla Tweaks）

自動下載材質包的主程式，會向 Vanilla Tweaks 網站產生並下載指定的材質包。

- 啟動時會從網站讀取目前有的版本並列出來，網站新增版本不用改程式。
- 輸入版本：不輸入、直接按 Enter 就是最新版本；按 Q 取消。格式不對或網站沒有的版本會請你重新輸入。
- 小版本會自動歸到大版本：1.21.1 ～ 1.21.11 都使用 1.21，26.3.1 使用 26.3。
- 依版本自動選用材質包清單：26.x 以後使用 `PACKS_26_3`（125 個），1.21 及更舊使用 `PACKS_1_21_11`（122 個）。兩份清單分別取自 TaurusTweaks 26.3 與 1.21.11 材質包 ZIP 內的 `Selected Packs.txt`。
- 某個版本網站還沒有的材質包會列為「尚未更新」，不算錯誤。
- 檔案儲存為 `~/Downloads/VanillaTweaks_MC<輸入的版本>.zip`。
- 想增減材質包，直接修改檔案中的清單即可（名稱要用網站內部名稱，例如 `NoFog`）。

### 注意事項

- 這是個人的非官方工具，與 Modrinth、Vanilla Tweaks 沒有任何隸屬關係。
- 材質包下載器是模擬瀏覽器操作 Vanilla Tweaks 網站，網站改版時可能無法使用。請勿用迴圈或排程反覆下載，以免造成網站負擔。
- 下載到的是 Vanilla Tweaks 網站產生的 ZIP，材質包組合相同，但不包含 TaurusTweaks 作者自訂的名稱與圖示。

---

## 授權條款

本專案由作者自行撰寫的程式碼及相關文件，採用 **GNU General Public License v3.0 only（GPL-3.0-only）** 授權。完整授權條款請參閱專案根目錄中的 [LICENSE](LICENSE)。

你可以使用、研究、修改及散布本專案；若散布本專案或其衍生作品，必須遵守 GPL-3.0-only 的條款，包括提供相應原始碼，並依照相同授權條款散布受 GPL 規範的衍生作品。

### 第三方套件與外部內容

本專案不會將第三方套件、外部網站內容或其他作者的作品重新授權為 GPL-3.0-only。它們仍各自適用其原作者、維護者或來源所提供的授權條款及使用條款。

- **Python 套件及相依元件**：依各套件實際採用的授權條款。
- **Modrinth**：模組檔案及專案內容依各模組作者所指定的授權。
- **Vanilla Tweaks**：網站產生的材質包及相關素材依其原始授權與網站使用條款。
- **TaurusTweaks**：本專案參考其 ZIP 中的 `Selected Packs.txt` 整理材質包選擇清單；該清單及相關內容不因此被本專案重新授權。

散布本專案時，請另外確認實際隨附或下載的第三方元件、模組、材質包及資料之授權要求。
