# solana-stablecoin

基于 Solana 区块链的去中心化稳定币项目。

## 项目简介

本项目实现了一个运行在 Solana 区块链上的稳定币合约，支持用户铸造、赎回、转账稳定币，并可与其他 DeFi 协议集成。

## 功能特性

- 稳定币铸造与赎回
- 余额查询与转账
- 价格预言机集成（如有）
- 质押资产管理（如有）
- 安全性与权限控制

## 技术栈

- Solana Program Library (Rust)
- Anchor 框架（如有）
- 前端：React/Vue/Next.js（如有）
- 其他依赖：见 `Cargo.toml`/`package.json`

## 快速开始

### 环境准备

- Rust 1.60+
- Solana CLI
- Node.js 16+（如有前端）
- Anchor CLI（如用 Anchor）

### 部署合约

```bash
# 安装依赖
cargo build-bpf

# 部署到本地测试网
solana-test-validator

# 部署合约
solana program deploy target/deploy/solana_stablecoin.so
```

### 运行前端（如有）

```bash
cd frontend
npm install
npm run dev
```

## 参与贡献

欢迎提交 Issue 和 PR！请遵循以下流程：

1. Fork 本仓库
2. 新建分支
3. 提交更改
4. 发起 Pull Request

## 许可证

MIT License