# 麦当劳 MCP API 完整参考

## 认证
```
Authorization: Bearer NpxB1YORiqXFFmnV1hx8V2iNIOWGS37w
```
每分钟最多 600 次请求

---

## 1. delivery-query-addresses（查配送地址列表）

**必填参数**:
- `beType`: string, 麦乐送="2"，团餐="6"

**响应**:
```json
{
  "data": {
    "addresses": [{
      "addressId": "xxx",
      "contactName": "张三",
      "phone": "152****6666",
      "fullAddress": "xx省xx市xxx小区 x栋x单元xxx室",
      "storeCode": "12345",
      "storeName": "xxx餐厅",
      "beCode": "12345"
    }]
  }
}
```

---

## 2. delivery-create-address（新增配送地址）

**必填参数**:
- `city`: 城市名，如 "杭州市"
- `contactName`: 联系人姓名
- `phone`: 11位纯数字手机号
- `address`: 配送地址（不含门牌号），如 "拱墅区中环大厦"
- `addressDetail`: 门牌号，如 "704"
- `beType`: "2"（麦乐送）或 "6"（团餐）

**响应**:
```json
{
  "success": true,
  "data": {
    "addressId": "xxx",
    "storeCode": "3330445",
    "beCode": "333044502",
    "storeName": "麦当劳杭州沈塘桥地铁A出口餐厅"
  }
}
```

---

## 3. query-meals（查询菜单）

**必填**: storeCode, orderType
**可选**: beCode

- orderType: "1"=到店, "2"=外送
- beCode: delivery-order 中获取，非外送场景可省略

**餐品编码来自**: `data.categories[].meals[].code`
**名称和价格来自**: `data.meals[code].name` / `data.meals[code].currentPrice`

---

## 4. calculate-price（价格计算）

**必填**: storeCode, orderType, items
**可选**: beCode, addressId

```json
{
  "storeCode": "3330445",
  "beCode": "333044502",
  "orderType": "2",
  "addressId": "xxx",
  "items": [{"productCode": "501205", "quantity": 1}]
}
```

**响应**:
```json
{
  "data": {
    "productList": [{"productName": "中薯条", "quantity": 1}],
    "productPrice": 1550,
    "deliveryPrice": 600,
    "packingPrice": 80,
    "price": 2230
  }
}
```
price 单位为分

---

## 5. create-order（创建订单）

**必填**: storeCode, orderType, items
**外送必填**: addressId

```json
{
  "storeCode": "3330445",
  "beCode": "333044502",
  "orderType": "2",
  "addressId": "1036415930195656861676122817",
  "items": [{"productCode": "501205", "quantity": 1}]
}
```

**成功响应**:
```json
{
  "data": {
    "orderId": "1030493870000764476903783230",
    "payH5Url": "https://m.mcd.cn/mcp/scanToPay?orderId=...",
    "orderDetail": {
      "storeName": "麦当劳杭州沈塘桥地铁A出口餐厅",
      "orderProductList": [{"productName": "中薯条", "quantity": 1, "price": "15.5"}],
      "totalAmount": "22.3",
      "realTotalAmount": "22.3",
      "deliveryInfo": {
        "deliveryAddress": "中环大厦",
        "addressDetail": "704",
        "customerNickname": "神奇海螺",
        "mobilePhone": "195****7680"
      }
    }
  }
}
```

---

## 6. query-order（查订单）

**必填**: orderId

---

## 7. query-nearby-stores（查附近门店）

**必填**: searchType, beType

- searchType: "1"=收藏门店, "2"=按位置搜索
- beType: "1"=到店搜索, "2"=外送搜索
- city: 城市名（searchType=2 时必填）
- keyword: 位置关键词（searchType=2 时必填）

⚠️ beType=2 时的搜索结果为外送可用门店

---

## 重要笔记

1. **address 和 addressDetail 必须分开**：不能把门牌号合并到 address 里
2. **beCode 不能为空**：外送场景 beCode 必须来自 delivery-query-addresses 返回值
3. **先算价再下单**：calculate-price 可以验证地址+门店是否匹配
4. **支付链接有效期**：订单创建后应立即推送支付链接
5. **同一地址可创建多次**：重复地址会生成新 addressId，但绑定同一门店
