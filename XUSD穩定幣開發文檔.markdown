# XUSD穩定幣開發文檔

## 目錄
1. [項目概述](#項目概述)
2. [技術架構](#技術架構)
   - [穩定幣創建](#穩定幣創建)
   - [核心功能](#核心功能)
     - [抵押功能](#抵押功能)
     - [兌換功能](#兌換功能)
     - [償還功能](#償還功能)
3. [業務流程](#業務流程)
4. [技術實現細節](#技術實現細節)
   - [前端實現](#前端實現)
   - [智能合約開發](#智能合約開發)
   - [SPL Token程序](#spl-token程序)
   - [預言機集成](#預言機集成)
5. [安全考慮](#安全考慮)
6. [後期擴展計劃](#後期擴展計劃)
7. [開發環境設置](#開發環境設置)
8. [部署與測試](#部署與測試)

## 項目概述
XUSD是一個在Solana區塊鏈上開發的穩定幣，與美元1:1錨定，旨在提供高效的抵押、兌換和償還功能。項目支持高性能交易並為後期功能擴展（如動態錨定、流動性池）預留接口。本文檔詳細介紹技術實現、業務流程和部署步驟。

## 技術架構

### 穩定幣創建
- **目標**：在Solana上創建XUSD穩定幣，實現與美元1:1錨定。
- **實現方式**：
  - 使用Solana SPL Token程序創建可流通代幣。
  - 配置代幣精度為6位小數（微美元級）。
  - 設置初始鑄幣權限，採用多重簽名治理結構。
- **依賴**：
  - Solana CLI（v1.18.0+）
  - Rust（v1.68+）
  - `spl-token`庫

### 核心功能

#### 抵押功能
- **功能描述**：
  - 用戶抵押SOL或其他白名單代幣，系統根據預言機價格發放XUSD。
  - 最低抵押比率為150%，以確保系統穩定性。
- **技術實現**：
  - 智能合約驗證抵押品價值，與預言機數據進行比較。
  - 集成Chainlink或其他Solana兼容預言機獲取實時價格。
  - 自動清算抵押比率低於130%的抵押品。
- **限制**：
  - 僅支持白名單代幣（如SOL、USDC）。
  - 最低抵押金額為等值100美元。

#### 兌換功能
- **功能描述**：
  - 用戶燒毀XUSD以兌換等值法幣。
- **技術實現**：
  - 通過支付網關（如Stripe或Circle）實現法幣兌換。
  - 智能合約處理XUSD燒毀並記錄交易。
- **要求**：
  - 完成KYC/AML合規檢查。
  - 確保法幣儲備充足並由第三方託管。

#### 償還功能
- **功能描述**：
  - 用戶償還XUSD以贖回抵押品。
- **技術實現**：
  - 智能合約驗證償還金額並解鎖對應抵押品。
  - 支持部分或全部償還，自動計算利息（若啟用）。
- **要求**：
  - 即時結算，確保抵押品快速解鎖。
  - 提供利息計算選項（默認0%）。

## 業務流程
以下為XUSD穩定幣的核心業務流程，使用Mermaid流程圖展示：

```mermaid
graph TD
    A[用戶] -->|抵押SOL或其他代幣| B{智能合約}
    B -->|驗證抵押品價值| C[發放XUSD]
    C -->|持有或使用| D[用戶]
    D -->|兌換法幣| E[燒毀XUSD]
    E -->|通過支付網關| F[獲得法幣]
    D -->|償還XUSD| G{智能合約}
    G -->|解鎖抵押品| H[用戶取回抵押品]
```

## 技術實現細節

### 前端實現
- **技術棧**：
  - React（使用CDN加載，版本18.x）。
  - Tailwind CSS（用於響應式設計）。
  - Solana Web3.js（與區塊鏈交互）。
- **功能模塊**：
  - **錢包連接**：支持Phantom、Solflare等錢包，通過`@solana/web3.js`實現。
  - **KYC認證**：簡單表單收集用戶信息，通過後端API提交KYC數據。
  - **抵押兌換**：展示實時預言機價格，允許用戶輸入抵押金額並獲取XUSD。
  - **法幣兌換**：生成二維碼，掃碼後通過支付網關完成法幣兌換。
- **示例代碼**（React組件）：
```jsx
import { useState } from 'https://cdn.jsdelivr.net/npm/react@18.2.0/+esm';
import { Connection, PublicKey } from 'https://cdn.jsdelivr.net/npm/@solana/web3.js@1.95.3/+esm';

const CollateralForm = () => {
  const [amount, setAmount] = useState(0);
  const connection = new Connection('https://api.devnet.solana.com');

  const handleCollateral = async () => {
    // 調用智能合約進行抵押
    console.log(`抵押 ${amount} SOL`);
  };

  return (
    <div className="p-4 bg-gray-100 rounded">
      <input
        type="number"
        value={amount}
        onChange={(e) => setAmount(e.target.value)}
        className="border p-2 mr-2"
        placeholder="輸入SOL數量"
      />
      <button
        onClick={handleCollateral}
        className="bg-blue-500 text-white p-2 rounded"
      >
        抵押並兌換XUSD
      </button>
    </div>
  );
};
```

### 智能合約開發
- **編程語言**：Rust（使用Anchor框架簡化開發）。
- **主要模塊**：
  - 代幣鑄造與燒毀：控制XUSD供應量。
  - 抵押品管理：鎖定和解鎖抵押品。
  - 預言機集成：獲取實時價格數據。
  - 清算邏輯：自動處理低抵押比率情況。
- **示例代碼**（簡化Anchor合約）：
```rust
use anchor_lang::prelude::*;
use anchor_spl::token::{Mint, Token, TokenAccount};

declare_id!("YourProgramIDHere");

#[program]
pub mod xusd {
    use super::*;

    pub fn initialize_collateral(ctx: Context<InitializeCollateral>, amount: u64) -> Result<()> {
        let collateral = &mut ctx.accounts.collateral;
        collateral.amount = amount;
        collateral.owner = *ctx.accounts.user.key;
        Ok(())
    }
}

#[derive(Accounts)]
pub struct InitializeCollateral<'info> {
    #[account(mut)]
    pub collateral: Account<'info, Collateral>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[account]
pub struct Collateral {
    pub amount: u64,
    pub owner: Pubkey,
}
```

### SPL Token程序
- **代幣創建**：
  - 使用`spl-token`命令創建XUSD代幣。
  - 配置代幣屬性：名稱為“XUSD”，符號為“XUSD”，精度為6。
- **代幣操作**：
  - 抵押時鑄造XUSD到用戶錢包。
  - 兌換或償還時燒毀XUSD。
  - 支持標準SPL代幣轉賬。
- **示例命令**：
```bash
spl-token create-token --decimals 6 --name XUSD --symbol XUSD
```

### 預言機集成
- **數據源**：Chainlink Solana預言機。
- **實現方式**：
  - 智能合約通過Chainlink接口獲取SOL/USD等價格數據。
  - 每5分鐘更新價格，異常價格觸發回退邏輯。
- **示例代碼**（偽代碼）：
```rust
fn fetch_price(oracle: &AccountInfo) -> Result<u64> {
    let price_data = oracle.get_price_data()?;
    if price_data.is_valid() {
        Ok(price_data.value)
    } else {
        Err(ProgramError::InvalidOracleData)
    }
}
```

## 安全考慮
- **智能合約審計**：聘請Quantstamp或CertiK進行全面審計。
- **多重簽名**：鑄幣權限需3/5多簽，防止單點故障。
- **預言機驗證**：集成多個預言機（如Chainlink、Pyth）進行價格交叉驗證。
- **緊急暫停**：設置全局暫停功能，僅限治理多簽觸發。
- **資金安全**：法幣儲備由獨立託管機構管理，定期公開審計報告。

## 後期擴展計劃
- **新增抵押品**：
  - 支持更多SPL代幣（如SRM、RAY）。
  - 探索NFT和跨鏈資產（如ETH通過Wormhole）。
- **可編程穩定幣**：
  - 動態調整錨定比率以應對市場波動。
  - 添加利息生成功能，分配給流動性提供者。
- **流動性池與AMM**：
  - 集成Raydium，創建XUSD/USDC交易對。
  - 實現自動做市商（AMM）以提高流動性。

## 開發環境設置
- **工具**：
  - Rust（1.68+）
  - Solana CLI（1.18.0+）
  - Anchor框架（0.30+）
- **依賴安裝**：
```bash
cargo install --git https://github.com/coral-xyz/anchor anchor-cli --locked
solana-install init 1.18.0
npm install @solana/web3.js @solana/spl-token
```
- **測試環境**：
  - 本地Solana測試網絡（`solana-test-validator`）。
  - 模擬Chainlink預言機數據。

## 部署與測試
- **部署步驟**：
  1. 編譯智能合約：`anchor build`。
  2. 部署到Solana Devnet：`anchor deploy`。
  3. 配置代幣和治理權限：`spl-token authorize`。
- **測試內容**：
  - 功能測試：
    - 抵押SOL並發放XUSD。
    - 兌換XUSD為法幣並燒毀。
    - 償還XUSD並贖回抵押品。
  - 壓力測試：模擬高並發抵押和兌換。
  - 異常測試：模擬價格劇烈波動和預言機失效。
- **測試工具**：
  - Anchor測試框架。
  - Mocha/Chai（前端測試）。