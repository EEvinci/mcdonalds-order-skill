---
name: mcdonalds-order
description: 麦当劳外卖点餐 Skill。当用户说「点麦当劳」「帮我点麦当劳」「麦当当」「mcdonalds」时激活。支持：查询菜单、创建订单、查询订单状态、配送地址管理（新增地址、查询地址列表）。基于麦当劳官方 MCP API，需要 Bearer Token 认证。
---

# 麦当劳外卖点餐 Skill

## 核心流程

```
确认地址 → 查询菜单 → 价格计算 → 创建订单 → 推送支付链接
```

## MCP API 调用规范

**Base URL**: `https://mcp.mcd.cn`
**认证**: `Authorization: Bearer {MCP_TOKEN}`（固定 Token）
**MCP Token**: `YOUR_MCP_TOKEN`

**所有请求格式**:
```python
payload = {'jsonrpc':'2.0','method':'tools/call','params':{'name':'{TOOL_NAME}','arguments':{...}}}
data = json.dumps(payload).encode()
req = urllib.request.Request(URL, data=data, headers={
    'Authorization': 'Bearer YOUR_MCP_TOKEN',
    'Content-Type': 'application/json'
})
with urllib.request.urlopen(req, timeout=15) as r:
    d = json.loads(r.read())
sc = d['result']['structuredContent']  # API 响应解析方式
```

## 工具速查

| 工具 | 用途 | 关键参数 |
|------|------|---------|
| `delivery-query-addresses` | 查地址列表 | **必填: beType=2** |
| `delivery-create-address` | 新增配送地址 | city, address, addressDetail, contactName, phone, **beType=2** |
| `query-meals` | 查询菜单 | storeCode, beCode, orderType=2 |
| `calculate-price` | 算价 | storeCode, beCode, addressId, orderType=2, items |
| `create-order` | 创建订单 | storeCode, beCode, addressId, orderType=2, items → 返回 payH5Url |
| `query-order` | 查订单详情 | orderId |

## 地址管理

### 查询用户配送地址
```
delivery-query-addresses + {beType: "2"}
```
响应中每个地址含: addressId, storeCode, beCode, fullAddress

### 创建新地址
```
delivery-create-address + {
    city: "杭州市",
    contactName: "姓名",
    phone: "11位手机号",
    address: "配送地址（如拱墅区中环大厦）",
    addressDetail: "门牌号（如704）",
    beType: "2"
}
```
⚠️ address 是配送地址（不含门牌），addressDetail 是门牌号（必填，分开传）

## 下单完整流程

### Step 1: 确认配送地址
优先用 `delivery-query-addresses` 查现有地址：
- 有现成地址 → 直接用 addressId + 对应的 storeCode/beCode
- 没有 → 用 `delivery-create-address` 新建

### Step 2: 查询菜单（如用户未指定餐品）
```
query-meals + {storeCode, beCode, orderType: "2"}
```
遍历 `data.categories[].meals[].code` 配合 `data.meals[code]` 获取名称和价格

### Step 3: 价格计算
```
calculate-price + {storeCode, beCode, addressId, orderType: "2", items: [{productCode, quantity}]}
```

### Step 4: 创建订单
```
create-order + {storeCode, beCode, addressId, orderType: "2", items: [{productCode, quantity}]}
```
成功响应含: orderId, payH5Url（支付链接）

### Step 5: 推送支付链接给用户
发送 payH5Url，点击即可支付

## 常见错误处理

| 错误信息 | 原因 | 解决 |
|---------|------|------|
| 城市不能为空 | delivery-create-address 缺少 city 或字段名错误 | 确认字段名为 `city` |
| 配送地址门牌号不能为空 | addressDetail 缺失 | 填入门牌号字段 |
| 无法配送到当前地址 | 门店不支持该地址 | 换用该地址在 delivery-query-addresses 中绑定的 storeCode/beCode |
| 麦乐送(beType=2)，仅支持到店取餐 | query-nearby-stores 用错 beType | 搜索外送门店用 beType=2，搜索结果中的门店均支持外送 |

## 详细工具参数参考

完整参数说明见: `references/mcp-api.md`
