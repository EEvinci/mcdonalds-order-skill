# 🍔 麦当劳外卖点餐 Skill

麦当劳外卖点餐 OpenClaw Skill，基于官方 MCP API，一句话完成点餐。

## 安装方式

**直接发送链接给 OpenClaw（推荐）：**

```
请安装 麦当劳点餐 Skill，帮我点麦当劳外卖。
技能地址：https://github.com/EEvinci/mcdonalds-order-skill
```

OpenClaw 自动识别 GitHub 链接并完成安装。

## 功能一览

| 功能 | 说明 |
|------|------|
| 📍 地址管理 | 查询已有地址、新增配送地址 |
| 🍟 菜单查询 | 查询当前门店可售餐品 |
| 💰 价格计算 | 下单前确认价格 |
| 🧾 创建订单 | 自动创建订单并推送支付链接 |
| 📦 订单查询 | 查看订单状态和详情 |

## 触发词

- 「点麦当劳」
- 「帮我点麦当劳」
- 「麦当当」
- 「mcdonalds」
- 「点个麦当劳外卖」

## 使用示例

```
你：帮我点一份中薯条送到XX
Kiko：收到！马上帮你下单
Kiko：✅ 订单创建成功！
     中薯条 ×1 | ¥22.30
     配送：[配送门店] → [配送地址]
     👉 点击支付：https://m.mcd.cn/mcp/scanToPay?orderId=...
```

## 工作流程

```
用户说「点麦当劳」或「帮我点麦当劳到XXX」
        ↓
1. 读取已有地址列表（delivery-query-addresses，beType=2）
        ↓
2. 有现成地址？→ 直接用
   没有/新地址？→ 调用 delivery-create-address 新增
        ↓
3. 查询菜单（用户未指定餐品时）
        ↓
4. 价格计算（calculate-price）
        ↓
5. 创建订单（create-order）
        ↓
6. 推送支付链接 → 用户点击支付
```

## 技术栈

- OpenClaw Skill
- 麦当劳官方 MCP API (`https://mcp.mcd.cn`)
- MCP Token: `NpxB1YORiqXFFmnV1hx8V2iNIOWGS37w`

## API 文档

完整参数说明见：[mcp-api.md](mcdonalds-order/references/mcp-api.md)

## 许可证

MIT
