# 喵喵怪游標 for Mac

[![macOS 驗證](https://github.com/fuwabaubau920525/xmgy671121/actions/workflows/macos-validate.yml/badge.svg)](https://github.com/fuwabaubau920525/xmgy671121/actions/workflows/macos-validate.yml)

把 7 組「喵喵怪」動畫游標（箭頭、等待、文字輸入、忙碌等）一鍵安裝到 macOS 的小工具。
雙擊開啟、按幾個鍵就完成，不需要打開終端機。

> Windows 版安裝程式在另一個資料夾 `app\`（本 repo 只放 Mac 版）。

## 直接下載

下載 [`mac/喵喵怪游標-for-Mac.zip`](mac/喵喵怪游標-for-Mac.zip)，解壓縮後把 **喵喵怪游標.app** 拖進「應用程式」資料夾。

## 安裝步驟

1. 解壓縮 `喵喵怪游標-for-Mac.zip`。
2. 把 `喵喵怪游標.app` 拖到 `/Applications`（應用程式）。
3. **第一次開啟前**，先移除隔離屬性，否則 macOS 會說 App「已損毀」：

   ```bash
   xattr -dr com.apple.quarantine "/Applications/喵喵怪游標.app"
   ```

   App 已經做過 **ad-hoc 簽章**（`codesign --sign -`），簽章本身是有效的；但沒有 Apple Developer ID、也無法公證，所以下載後仍會帶隔離屬性。

   或在「系統設定 → 隱私權與安全性」最下方按「仍要打開」。

4. 雙擊 `喵喵怪游標.app`，依畫面指示選擇要安裝的游標。
5. 安裝完成後到「系統設定 → 輔助使用 → 指標控制 → 游標大小」調整大小，效果最明顯。

### 系統需求

| 項目 | 需求 |
| --- | --- |
| macOS | 11.5（Big Sur）以上 |
| 晶片 | Apple Silicon 或 Intel 皆可 |
| 權限 | 安裝時會要求輸入管理員密碼（安裝 Mousecape 用） |

程式會自動判斷晶片與系統版本：

- **Apple Silicon + macOS 26 (Tahoe)**：使用相容的 Mousecape 1820 分支。
- **Intel 或較舊的 macOS**：使用 Mousecape 0.0.6。
- 若偵測到舊版 Mousecape 0.0.6 且系統是 macOS 26，會詢問是否自動替換。

## 還原成原本的游標

再次開啟 `喵喵怪游標.app`，選擇「還原原始游標」即可。
（底層是 `mousecloak -r`，會移除 Mousecape 套用的游標設定。）

## 內容物

```
mac/
├── 喵喵怪游標-for-Mac.zip      # 發佈用壓縮檔（含 .app）
├── 喵喵怪游標.app/             # 可直接雙擊的 App Bundle
│   └── Contents/
│       ├── MacOS/launcher      # Platypus ScriptExec（universal）
│       ├── Info.plist
│       └── Resources/
│           ├── script          # 安裝流程腳本（bash）
│           ├── capes/          # 7 組 .cape 游標主題
│           ├── Mousecape-*.zip # 兩種 Mousecape 版本
│           └── *.txt           # 授權與說明
├── capes/                      # 原始 .cape 檔
├── src/script.sh               # 腳本原始碼（與 App 內相同）
└── 使用說明.txt
```

7 組主題：`banana`、`dog`、`fairy`、`halloween`、`niuniu`、`run`、`wand`。

## 常見問題

**Q：為什麼一定要跑 `xattr`？**
A：App 只有 ad-hoc 簽章，沒有 Apple Developer ID，無法通過公證（Notarization）。從網路下載後 macOS 會加上隔離屬性，移除後即可正常開啟。若要做到「雙擊就開、不用任何指令」，需要付費的 Apple Developer Program 憑證。

**Q：開啟後沒反應？**
A：本 App 是 `LSUIElement`（不顯示 Dock 圖示），流程會在視窗中詢問，完成後自動結束。若完全沒視窗，請確認已執行上面的 `xattr` 指令。

**Q：游標沒有變？**
A：Mousecape 套用後，部分 App 需要重新啟動；也請確認「系統設定 → 輔助使用 → 指標控制」中的游標大小不是最小。

## 自動驗證（GitHub Actions）

每次推送 `mac/` 的變更，GitHub 會在雲端的 macOS 虛擬機上自動執行
[`.github/workflows/macos-validate.yml`](.github/workflows/macos-validate.yml)：

- 檢查 `Info.plist`、`AppSettings.plist` 與 7 組 `.cape` 格式
- 確認 `launcher` / `script` 有執行權限，且 `launcher` 是 arm64 + x86_64 universal
- 用 `codesign --verify --deep --strict` 驗證 ad-hoc 簽章
- 確認 `script` 語法正確、純 LF，且與 `src/script.sh` 一致
- 驗證發佈 zip 的完整性、權限，並與 `喵喵怪游標.app` 逐檔比對
- 用假的 `osascript` / `Mousecape` 在無介面環境跑一次「安裝 → 還原 → 移除」

> 雲端 macOS 沒有圖形桌面可以操作，這是自動化驗證，不是遠端桌面。

## 授權與致謝

- 游標美術：本專案作者。
- [Mousecape](https://github.com/alexzielenski/Mousecape)（Alex Zielenski，MIT）與 [Tahoe 相容分支](https://github.com/AdamWawrzynkowskiGF/Mousecape-TahoeSupport)。
- [Platypus](https://sveinbjorn.org/platypus)（Sveinbjörn Þórðarson）用於產生 App 啟動器。
- 各授權全文見 `mac/喵喵怪游標.app/Contents/Resources/` 內的 `*-LICENSE.txt` 與 `*-NOTICE.txt`。
