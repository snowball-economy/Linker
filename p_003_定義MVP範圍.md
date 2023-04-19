# 定義 MVP 範圍

## 最小完成項目基準
- 一個 Dapp 網頁
  - Page
    - POAP new
      - 用戶可以上傳照片並生成 POAP 智能合約
      - 可以用網頁生成 QR code
        - 可以設定 QR code 最多能 mint 的數量 (ex.1)
          - 如果超過 mint 數量,跳轉到網頁後,會顯示 mint 數量已經用完
    - POAP detail
      - 用手機掃描後可以自動跳轉到 Dapp 網頁,並且 mint POAP
      - Owner
        - 可以看到使用者頁面
        - 可以看到擁有者頁面
          - Key 列表 (Key 就是 QR code 的內容)
            - name
            - 已 mint 數量 / 總 mint 數量
            - 顯示 QR code
      - User
        - 可以看到目前這個 POAP 的詳情
          - POAP 資料
          - 鏈上數據
            - mint 數量
            - mint 人數
            - mint 紀錄
    - User Page (Optional)
      - 自己持有的 POAP
      - 可以創建資料夾來分類自己的 POAP

------------------------

實現一次性 QR code 方式：
- 中心化：
  - 中心化資料庫存儲,並且只能用我們 APP 去讀取,讓 POAP 能 mint 的 role 設定為我們平台的帳戶
- 去中心化：
  - 合約生成 Key

### 中心化

- 建立中心化伺服器：在後端建立一個中心化的伺服器，用於管理所有生成的QR code和對應的mint限制。當用戶通過Dapp創建新的POAP時，將QR code的相關信息（如mint限制）儲存在伺服器上。
- 生成QR code：在伺服器生成一個唯一的QR code ID，並將其與伺服器的API端點一起編碼到QR code中。您可以使用一個QR code生成庫來實現這一功能。
- Mint POAP：當用戶掃描QR code並進入Dapp網頁時，解析QR code中的API端點和QR code ID。通過API端點向伺服器發送請求，並傳遞QR code ID。在後端，檢查對應的QR code ID的mint數量限制。如果尚未達到限制，則調用智能合約的mint函數創建新的POAP，並更新伺服器上的mint計數。
- 檢查mint限制：在後端檢查mint限制，並根據檢查結果向前端返回相應的響應。如果達到限制，則拒絕mint操作，並顯示相應的錯誤消息。否則，完成mint操作並更新mint計數。

此方法的優點是更容易實現和擴展，因為伺服器可以輕鬆管理和更新QR code及其相關的限制。然而，這種方法引入了中心化的單點故障，可能會影響去中心化應用的安全性和可靠性。

### 去中心化

智能合約設計：在POAP智能合約中，為每個QR code生成唯一的ID（例如，使用UUID或哈希函數）。在合約中設置一個映射，將QR code ID映射到它的mint數量限制，並確保只能使用一次。

```solidity
mapping(bytes32 => uint256) public qrCodeToMintLimit;
mapping(bytes32 => uint256) public qrCodeToMintCount;
```

生成QR code：當用戶在網頁上生成一個新的POAP時，生成一個唯一的QR code ID，並將其與POAP智能合約地址一起編碼到QR code中。您可以使用一個QR code生成庫來實現這一功能。
Mint POAP：當用戶掃描QR code並進入Dapp網頁時，解析QR code中的智能合約地址和QR code ID。在後端，檢查對應的QR code ID的mint數量限制。如果尚未達到限制，則調用智能合約的mint函數創建新的POAP。

```solidity
function mintPOAP(address to, bytes32 qrCodeID) public {
    require(qrCodeToMintCount[qrCodeID] < qrCodeToMintLimit[qrCodeID], "Mint limit reached.");
    // Mint the POAP token
    // Update the mint count
    qrCodeToMintCount[qrCodeID]++;
}
```

檢查mint限制：在mint函數中，檢查與QR code ID相關的mint計數是否已達到限制。如果達到限制，則拒絕mint操作，並顯示相應的錯誤消息。否則，完成mint操作並更新mint計數。
通過以上方法，您可以實現生成只能用來mint一次POAP的連結。在後續的開發中，您可以根據需求為QR code ID添加更多的限制條件，例如設置有效期或添加用戶驗證等。