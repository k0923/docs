# RESTful API 接口设计规范

## 1. 核心原则
*   **资源导向**：URL 表示资源（名词），HTTP 方法表示操作（动词）。
*   **复数优先**：除非资源是全局唯一的（如 `/me`），否则一律使用复数名词。
*   **统一格式**：请求和响应均使用 JSON 格式。

## 2. URL 路径设计 (Routing)

### 2.1 命名规范
*   **使用名词**：路径中**不得**包含动词（如 `get`, `create`, `add`, `delete`）。
*   **使用复数**：资源名称统一使用复数形式。
*   **分隔符**：URL 路径中使用中划线（kebab-case）分隔单词，尽量不使用下划线或驼峰。
    *   ✅ `/api/user-orders`
    *   ❌ `/api/userOrders`
    *   ❌ `/api/user_orders`
*   **小写**：URL 全部使用小写字母。

### 2.2 层级结构
对于存在层级关系的资源，使用嵌套路径，但建议**不超过 3 层**。

*   `GET /users/{id}/orders`：获取指定用户的订单列表。
*   `GET /shops/{id}/products`：获取指定店铺的商品。

### 2.3 版本控制
API 必须包含版本号，以便未来迭代而不影响旧版客户端。
*   格式：`/api/v{n}/resource`
*   示例：`/api/v1/users`

## 3. HTTP 方法与语义 (Methods)

动作由 HTTP Method 定义，严禁在 URL 中体现。

| 操作类型 | HTTP 方法 | URL 示例 | 状态码 (成功) | 备注 |
| :--- | :--- | :--- | :--- | :--- |
| **创建** | **POST** | `/api/v1/users` | **201 Created** | 对应新增数据 |
| **查询列表** | **GET** | `/api/v1/users` | **200 OK** | 支持分页、过滤 |
| **查询单条** | **GET** | `/api/v1/users/{id}` | **200 OK** | |
| **全量更新** | **PUT** | `/api/v1/users/{id}` | **200 OK** | 替换整个资源对象 |
| **部分更新** | **PATCH** | `/api/v1/users/{id}` | **200 OK** | 仅修改部分字段 |
| **删除** | **DELETE** | `/api/v1/users/{id}` | **204 No Content** | 或者是 200 OK |

## 4. 特殊场景路由规范

### 4.1 单例资源 (Singleton)
如果资源在全局只有一个（例如当前登录用户），可以使用单数。
*   `GET /api/v1/me` (获取当前用户信息)
*   `PUT /api/v1/me/password` (修改当前用户密码)

### 4.2 无法映射为资源的动作
如果操作确实无法用 CRUD 描述（例如“登录”、“计算”、“搜索”），可以使用动词作为子资源，但要保持克制。
*   `POST /api/v1/auth/login` (登录)
*   `POST /api/v1/users/{id}/disable` (禁用用户，比 PATCH 更语义化时使用)
*   `GET /api/v1/search?q=keyword` (搜索)

## 5. 请求与响应规范

### 5.1 数据格式
*   `Content-Type` 必须为 `application/json`。
*   字段命名：统一使用 **小驼峰命名法 (camelCase)**。
    *   ✅ `userName`, `createdAt`
    *   ❌ `user_name`, `created_at`

### 5.2 统一响应结构
建议后端所有接口返回统一的数据包裹结构：

```json
{
  "code": 200,          // 业务状态码 (0或200表示成功)
  "message": "success", // 提示信息
  "data": {             // 实际数据载荷
    "id": 1,
    "name": "Admin",
    "email": "admin@example.com"
  }
}
```

### 5.3 过滤、排序与分页
不要放在 URL 路径中，使用 **Query Parameters (查询参数)**。

*   **分页**：`GET /users?page=1&pageSize=10`
*   **排序**：`GET /users?sort=-created_at` (负号表示倒序)
*   **过滤**：`GET /users?status=active&role=admin`

## 6. 实战对比 (Do's and Don'ts)

| 场景 | ✅ 正确示范 (推荐) | ❌ 错误示范 (禁止) |
| :--- | :--- | :--- |
| **创建订单** | `POST /orders` | `POST /createOrder`<br>`POST /order/add` |
| **获取订单详情** | `GET /orders/1001` | `GET /order/1001`<br>`POST /getOrder?id=1001` |
| **删除产品** | `DELETE /products/50` | `GET /products/delete/50`<br>`POST /deleteProduct` |
| **获取特定用户的博客** | `GET /users/1/blogs` | `GET /blogs?userId=1` (也能用，但不推荐作为主路由) |

---

### 示例代码 (以创建数据为例)

**请求 URL:** `POST https://api.example.com/api/v1/articles`

**请求体 (Request Body):**
```json
{
  "title": "RESTful API 规范",
  "content": "这是内容...",
  "authorId": 123
}
```

**响应体 (Response Body):**
```json
{
  "code": 200,
  "message": "创建成功",
  "data": {
    "id": 456,
    "title": "RESTful API 规范",
    "createdAt": "2023-10-27T10:00:00Z"
  }
}
```

## DTO命名
### 1. DTO (Data Transfer Object) - 请求与响应
为了符合 **IDE 排序友好** 和 **资源优先** 的原则，推荐 **[Resource][Action][Suffix]** 的格式。

| 类型 | 命名模式 | ✅ 示例 | 说明 |
| :--- | :--- | :--- | :--- |
| **创建请求** | `[Resource]CreateRequest` | `DatasetCreateRequest` | 包含创建所需的字段 |
| **更新请求** | `[Resource]UpdateRequest` | `DatasetUpdateRequest` | 包含更新所需的字段 |
| **列表查询** | `[Resource]QueryRequest` | `DatasetQueryRequest` | 包含分页、筛选条件 |
| **通用响应** | `[Resource]Response` | `DatasetResponse` | 返回给前端的完整对象 |
| **列表响应** | `[Resource]ListResponse` | `DatasetListResponse` | 这种通常不需要，直接返回 Array 或 Page 对象即可 |













