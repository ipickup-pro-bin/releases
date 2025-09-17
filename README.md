# iPickup Pro 使用說明


## 啟動程式的步驟

1.  根據你的作業系統和 CPU 架構，下載合適的 iPickup Pro 壓縮檔案 <https://github.com/ipickup-pro-bin/releases/releases/latest>
2.  將壓縮檔解壓後，先執行 `run-config` 以開啟設定介面的 web UI，瀏覽器會自動打開設定介面的網址: <http://localhost:9999>
3.  設定完成後，再執行 `run` 即可進入自動搶購的操作
4.  程式啟動後，可於瀏覽器打開 dashboard，預設網址為 <http://localhost:3000> (位址中的 port 可於設定介面中修改)


## 設定說明

-   **General** (一般設定)
    -   一般設定包含 `Bot 註冊碼` 和 `Dashboard Port`
-   **Profiles** (取貨人資料)
    -   `Profile ID` - 自定意的 ID 以作辨認
-   **付款卡**
    -   `付款卡 ID` - 自定意的 ID 以作辨認
    -   `有效期至` - 付款卡到期日，格式 `MM/YY`
-   **Proxy 列表**
    -   `Proxy 列表 ID` - 自定意的 ID 以作辨認
    -   `Proxy 列表` - 每一行一個 proxy 位址，並跟隨此格式 `[USERNAME:PASSWORD@IP:PORT]` 或 `[IP:PORT:USERNAME:PASSWORD]`
-   **監察器**
    -   `檢測間距` - 檢測間距毫秒(ms)
    -   `型號` - 監察的型號，如沒提供，則會監察 `properties.yml` 內的所有型號
-   **任務** (程式執行時的搶購任務)
    -   任務由 `任務組` 和 `子任務` 組成
    -   每個 `任務組` 必需包含 **最少一個** `子任務`
    -   `任務組` 的作用是提供預設值給 `子任務` ，以便快速修改設定
    -   `子任務` 會繼承 `任務組` 的所有欄位值成預設值，並可逐一欄位進行覆寫
    -   所有欄位均非必需填，唯 `型號` 必需揀選一款
        -   欄位空白時會使用程式預設值或隨機值
-   **操作**
-   **訂單通知**
    -   `Discord` - 如需使用 Discord 通知，請參照[此連結](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks)以設立 Discord webhook


## 進階操作

```sh
$> ./ipickup_pro-linux-x64 --help
  -y, --no-confirmation  No confirmation
      --config           Config UI
  -p, --port PORT        Port number
      --unbind           Unbind license key
  -h, --help             Help
```


# 常見問題 / FAQ


### config.yml 不能讀取

請檢查 YAML 格式是否正確 <https://codebeautify.org/yaml-validator>


### 如何加入其他型號/機款？

可添加至 `properties.yml` 的 `models` 如下:

```yaml
model-id: # 自定意的型號ID
  sku: SKU # Apple商品 part number / SKU
  name: NAME # 商品名稱
```


### 下載新版本後，如何將設定保留?

將新的執行檔(e.g. .exe)複制至舊資料夾中，或將舊的 `config_override.yml` 複制至新資料夾中即可。


### Console 顯示的中文字為亂碼

-   Windows 10
    -   Control Panel -> Region

![img](https://i.imgur.com/XVzgFyb.png)

![img](https://i.imgur.com/jT3pNVc.png)


### **如何保存 log 或將程式放置於背後運作**

建議使用 [PM2](https://pm2.keymetrics.io/) ，於 bot 應用程式的 folder 內新增 `ecosystem.config.js` 內容如下：

```javascript
module.exports = {
  apps : [{
    name   : "ipickup-pro",
    script : "./ipickup_pro-linux-x64 -y",
    "args": [
      "--color"
    ],
    "env": { "DEBUG_COLORS": true }
  }]
}
```

執行輸入 `pm2 start` 即可。


### 監察器停止運作

如果監察器內顯示的時間在檢測間距過後一直沒有跳動，便是監察器停止了運作。

-   **解決方法 1 - 手動**
    
    打開 **Dashboard** > 按一下 **重啟監察器**

-   **解決方法 2 - 自動 (感謝 tg@singdad 提供)**
    -   使用 Tampermonkey 瀏覽器插件 (<https://www.tampermonkey.net/>)
        
        ```javascript
        // ==UserScript==
        // @name         iPickup Pro monitor auto-restart
        // @namespace    http://tampermonkey.net/
        // @version      0.1
        // @description  Check if an element's content is unchanged for 1 minute, then click a button
        // @author       You
        // @match        http://localhost:5555/*
        // @grant        none
        // ==/UserScript==
        
        (function() {
         'use strict';
        
         const checkTimeInterval = 60000;
        
         // Function to check the element and click the button if needed
         function checkElementAndClickButton() {
          const labelElement = document.querySelector('.ui.small.label');
          const buttonElement = document.querySelector('#app > div > div:nth-child(2) > div > button');
        
          if (!labelElement || !buttonElement) {
           console.error('Required elements not found');
           return;
          }
        
          let lastContent = labelElement.innerHTML;
        
          const checkMonitor = () => {
           const currentContent = labelElement.innerHTML;
           if (currentContent === lastContent) {
            buttonElement.click();
           } else {
            lastContent = currentContent;
           }
          };
        
          const intervalCheckMonitor = () => setInterval(checkMonitor, checkTimeInterval);
          setTimeout(intervalCheckMonitor, checkTimeInterval);
         }
        
         setTimeout(checkElementAndClickButton, 1000);
        })();
        ```
    
    -   用以下 PowerShell script 代替 `run.bat` (Windows 適用)
        
        ```shell
        # Define the path to the program and the log files
        $programPath = "C:\Users\singdad\Downloads\ipickup\ipickup_pro-win-x64.exe"
        $stdoutLogFilePath = "C:\Users\singdad\Downloads\ipickup\stdout.log"
        $stderrLogFilePath = "C:\Users\singdad\Downloads\ipickup\stderr.log"
        
        # Start the program and redirect its output to the log files
        $process = Start-Process -FilePath $programPath -RedirectStandardOutput $stdoutLogFilePath -RedirectStandardError $stderrLogFilePath -PassThru
        
        # Wait for the program to start and send an initial Enter key press
        Start-Sleep -Seconds 2
        Add-Type -AssemblyName System.Windows.Forms
        [System.Windows.Forms.SendKeys]::SendWait("{ENTER}")
        
        # Function to check if the process has output
        function HasOutput {
         $stdoutOutput = Get-Content -Path $stdoutLogFilePath -Tail 1
         $stderrOutput = Get-Content -Path $stderrLogFilePath -Tail 1
         return -not ([string]::IsNullOrWhiteSpace($stdoutOutput) -and [string]::IsNullOrWhiteSpace($stderrOutput))
        }
        
        # Monitor the output
        $lastOutputTime = Get-Date
        while (-not $process.HasExited) {
         if (HasOutput) {
          $lastOutputTime = Get-Date
         } elseif ((Get-Date) - $lastOutputTime -gt (New-TimeSpan -Minutes 2)) {
          [System.Windows.Forms.SendKeys]::SendWait("{ENTER}")
          $lastOutputTime = Get-Date
         }
         Start-Sleep -Seconds 10
        }
        
        # Clean up
        $process.WaitForExit()
        ```


### 常見的錯誤和處理方法

-   **訂單只能送貨**
    -   Apple Store 的風控管理，可調試不同的 delay 和開啟 `safe-mode` ，或使用 `browser-mode` (不建議，因 `browser-mode` 太慢和佔用較多資源)
    -   可使用[虚假信用卡](https://saijogeorge.com/dummy-credit-card-generator/)來測試有貨的商品，如設定能通過風控，會出現 card declined 的 error

-   **監察器未能獲取訊息**
    
    如果在 console 和 dashboard 的監察 bar 上持續顯示此訊息，則表示 IP ban，需切換 IP/proxy 或將 `interval` 的數值調高。

-   **登錄失敗，即將重試 / 不能打開頁面 / Server 未能接收請求, 即將重新提交&#x2026;**
    -   IP ban，需切換 IP/proxy
    -   如沒有足夠的 IP/proxy，則減少 Tasks 數量


### 問題回報及一般討論

TG 群組: [iPickup Pro Chat 2024](https://t.me/+7t9rErmqZJQ2ODI1)
