# 掌上莲峰（zslf）API 收集

[![OpenAPI](https://img.shields.io/badge/OpenAPI-3.0.3-green.svg)](https://spec.openapis.org/oas/v3.0.3)
[![Endpoints](https://img.shields.io/badge/endpoints-89-blue.svg)](#-覆盖情况)
[![Verified](https://img.shields.io/badge/实测验证-87%2F89-brightgreen.svg)](#-覆盖情况)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

电子科技大学中山学院「掌上莲峰」App 的完整 HTTP API 集合，**OpenAPI 3.0.3** 格式。

> 本文档的所有结论均经过**对生产后端的真实请求验证**，不是纯静态推断。

---

## 📊 覆盖情况

| 指标 | 数值 |
|---|---|
| 收录接口 | **89** 个 path / method |
| 已实测验证 | **87** 个（97.8%） |
| 未实测 | 2 个（`/member/user/*`，预留死接口，全包零调用点） |
| 接口分组 | **25** 个 |

---

## 🚀 快速上手

### 第 1 步：登录拿 token

```http
POST https://zslf.zsc.edu.cn/app/login?username=<学号>&password=<口令>
```

响应 `data.accessToken` 就是 token（**UUID v4 格式**）。

### 第 2 步：带上 token 调其它接口

```http
GET https://zslf.zsc.edu.cn/app/appUser/getUserInfo
Authorization: <accessToken>
```

就这一个头。**不用加 `Bearer ` 前缀**，服务端只认裸值。

### curl 示例

```bash
# 1. 登录
curl -X POST 'https://zslf.zsc.edu.cn/app/login?username=YOUR_ID&password=YOUR_PASS'

# 2. 查用户信息
curl 'https://zslf.zsc.edu.cn/app/appUser/getUserInfo' \
  -H 'Authorization: <accessToken>'

# 3. 查未读消息
curl -X POST 'https://zslf.zsc.edu.cn/app/appNotice/unreadNumber' \
  -H 'Authorization: <accessToken>'
```

---

## 📐 通用约定

### Base URL

| 用途 | Base |
|---|---|
| 绝大多数业务接口 | `https://zslf.zsc.edu.cn/app` |
| **`/run/*` 一组（运动 / 场馆 / 体测）** | `https://zslf.zsc.edu.cn`（**没有 `/app` 前缀**） |

> 最容易踩的坑：`/run/errand/getErrandList` 的完整地址是
> `https://zslf.zsc.edu.cn/run/errand/getErrandList`，**不是** `/app/run/...`。

### 鉴权

| 项 | 约定 |
|---|---|
| Header | `Authorization: <accessToken>` |
| 格式 | **裸 token，无 `Bearer ` 前缀** |
| token 形态 | UUID v4，**不是 JWT** |
| 服务端校验 | **严格**，5 种无效 token 实测全部返回 **HTTP 401** |
| 刷新机制 | **不存在**。登录响应里没有 `refreshToken` 字段，也没有刷新端点；过期只能重新登录 |
| 请求头宽松度 | 实测**只需 `Authorization` 一个头**；`Accept` / `platform` / `appVersion` / `Content-Type` 全部可选且不校验值 |

### 参数位置

| 位置 | 适用范围 |
|---|---|
| **URL query string** | 绝大多数接口（登录、改密、列表查询…） |
| **JSON body** | 仅 5 个：`/run/errand/{quickSave,syncSportData,reportLocation,addErrandTrackByData}`、`/mobileLogReport/addLog` |
| **multipart/form-data** | 仅上传接口 |

### 响应信封

```json
{
  "code": 200,
  "data": {},
  "message": "获取用户信息成功",
  "success": true
}
```

| 字段 | 说明 |
|---|---|
| `code` | **业务状态码**，`200` = 成功 |
| `data` | 业务数据；**失败时是空字符串 `""`**（不是 `null`） |
| `message` | 中文提示 |
| `success` | 与 `code` 语义一致，客户端从不读取 |

### ⚠️ 两层错误处理（必读）

```text
HTTP 401  →  未登录 / token 失效          → 跳登录页
HTTP 200  →  再看 body.code
              code === 200  →  成功
              code !== 200  →  失败，展示 body.message
```

**业务错误仍然是 HTTP 200。** 只判 HTTP 状态码会漏掉所有业务错误。

---

## 📚 接口分组

| 分组 | 说明 |
|---|---|
| `auth` | 登录鉴权 |
| `appUser` | 用户资料、头像、密码 |
| `appNotice` | 站内通知与未读数 |
| `announcement` | 校园公告 |
| `appFind` | 发现页菜单与配置字典 |
| `appCmsColumn` | CMS 栏目 |
| `appCmsArticle` | CMS 文章与详情 |
| `appCmsArticleLike` | 文章点赞 |
| `adApi` | 首页轮播图 |
| `business` | 周边商家与活动 |
| `appRecruit` | 兼职招聘 |
| `appRecommend` | 广场帖子与推荐 |
| `recycle` | 二手物品（闲置） |
| `questionNaire` | 调查问卷 |
| `imChat` | 在线客服 |
| `AppOperator` | 校园宽带套餐 |
| `appFeedback` | 意见反馈 |
| `mobileLogReport` | 客户端日志上报 |
| `run` | 运动 / 场馆预约 / 体测（base 无 `/app` 前缀） |
| `sign` | 签到与积分 |
| `verification` | 校园网认证 |
| `campusNetwork` | 校园网 / 宽带账号 |
| `upload` | 文件上传 |
| `version` | 版本更新 |
| `member` | 新版用户中心（预留，未启用） |

---

## 🔗 常用调用链

### 校园动态（首页）

```text
POST /app/appCmsColumn/getCmsByModule?name=学校动态
  → data.cmsColumn.id
POST /app/appCmsArticle/getCmsArticleByColumnId?cmsCloumnId=<id>&offset=0&limit=10
  → data.cmsArticleList[]
POST /app/appCmsArticle/articleDetailsById?articleId=<id>
  → data.articleDetails.content（HTML 正文）
```

### 签到积分

```text
POST /app/sign/getUserSign                          近 7 天签到日历
POST /app/sign/getSumIntegral                       总积分
POST /app/sign/getIntegralDetailByUserId?userId=<uid>  积分明细
```

### 校园跑

```text
POST /run/errand/getErrandList?userId=<uid>&offset=0&limit=10&isFinish=0
POST /run/errand/getTrackByTrid?trid=<trid>         轨迹点
POST /run/errand/syncSportData                      JSON body，必填 trid
```

### 功能配置

```text
POST /app/appFind/getDictTips?code=schoolRun&num=1           校园跑开关
POST /app/appFind/getDictTips?code=attendanceRadius&num=1    签到半径（实测 40 米）
```

---

## ⚠️ 已知服务端问题

以下为实测发现的服务端行为，复刻客户端时需要自行规避：

| 问题 | 影响接口 |
|---|---|
| 空参直接返回 **HTTP 500**（无参数校验） | `/sign/addUserSign`、`/run/site/addApplySelectTime`、`/run/site/getApplyDetail` |
| 异常栈直接返回给客户端 | `/run/physical/getPhysicalTestList`、`/run/site/addSiteApplyRelated` |
| `POST /appUser/resetNewPassWord` **不需旧密码 / 验证码** | 仅凭 `userId` 即可改密 |
| `POST /campusNetwork/getOperatorAccountByIDCard` **无参即返回宽带账号** | PII 泄露 |
| 口令明文走 URL query | `/login`、`/appUser/modifyPassword`、`/appUser/resetNewPassWord` |
| 响应键名拼写错误 | `getInternetInfo` 返回 `massage`；`getRecycleTypeList` 等响应键带 `get` 前缀 |
| 参数名拼写错误 | `cmsCloumnId`（少一个 n）、`addApplyDenfition` |

---

## 🖥 服务端技术栈

从异常栈泄漏信息得出：

| 组件 | 说明 |
|---|---|
| 应用服务器 | Apache Tomcat，`/usr/local/tomcat/webapps/ROOT` |
| 框架 | Spring + MyBatis |
| 连接池 | Alibaba Druid（`maxActive 50`） |
| 脚手架 | **Guns**（`cn.stylefeng.guns`） |

---

## 📦 在 Apifox 中使用

1. Apifox → 项目设置 → 导入数据 → OpenAPI
2. 选择本仓库的 [`openapi.yaml`](openapi.yaml)
3. 导入后自动得到 25 个分组、89 个接口，每个接口都带真实响应示例

发布公开文档站：Apifox 左侧「分享文档」→「发布文档站」。

---

## 🔬 数据来源

| 项 | 说明 |
|---|---|
| 分析对象 | `zslf.apk`（sha256 `416b4524321d6ab1dc6884b230b4fffd87fbf594143f99345abf2f815253fc92`） |
| 版本 | `1.0.83` / versionCode `10089`，包名 `com.zx.slm` |
| 客户端形态 | DCloud uni-app（Vue 3）混合应用，业务逻辑 100% 在 JS |
| 方法 | APK 拆包 + 18,400 个反编译 Java 类分析 + JS bundle 分析 + **生产后端真实请求实测** |
| 标注 | 每个接口的 description 里标明是否已实测 |

完整分析文档集见 [github.com/better-zslf](https://github.com/better-zslf)。

---

## ⚖️ 免责声明

本项目仅收录**接口契约信息**（路径、方法、参数、响应结构），用于客户端开发参考与技术研究。

- 不含任何用户凭据、token 或个人身份数据
- 请遵守目标服务的使用条款，勿用于滥用或攻击
- 使用本项目产生的一切后果由使用者自行承担

## 📄 License

[MIT](LICENSE)
