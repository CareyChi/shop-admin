# shop-admin · 管理员后台 · 前端开发文档

版本 1.0.0；日期 2026-09-27；接口合同 1.0.0。

本次仅编写文档。功能状态：⚪ 待实现；✔ 已实现。本文所有功能均为 ⚪，未编写或运行应用代码。

先阅读 [共同架构与约定](COMMON_ARCHITECTURE.md)，再按本文模块开发。前后端相同编号代表同一业务能力，但实现状态独立维护。示例均为虚构演示数据，不是生产账户或实际响应。

本仓库后端模块规划：admin-api、identity-service。所有对外接口以下述路径为相对地址，统一前缀 `/api/v1/admin`；完整路由由网关转发至本端 API。

## 前端结构及实现约束

所有后续前端程序放在 frontend/：src/api（fetch 与 DTO）、src/stores（Pinia）、src/router、src/views（按 A～F 模块组织）、src/components、src/composables、src/types、src/utils、src/styles；测试放 frontend/tests。当前只保留目录占位，不生成上述源码。

使用 Vue SFC + Composition API + TypeScript strict；组件只负责展示与交互，API 请求在 api 层，跨路由共享状态放 Pinia，表单局部状态用 ref/reactive。authStore 保存身份与权限，其他领域 store 以当前账户隔离，退出、换账户后重置。用户可修改的 tenantId 仅可做筛选，不成为授权依据。

每页都实现加载、空、失败、无权限四种反馈。表格默认 20 条、服务端分页；筛选变化回第一页，取消旧请求或比较请求序号；表单按钮防连点不是后端幂等的替代。商品/备注以文本插值展示，禁止直接 v-html。金额字符串转整数分计算展示，超出安全整数使用 BigInt/字符串格式化，不能 parseFloat 累加作为结算依据。

接口示例中的 data 为统一响应壳内字段；错误按 COMMON_ARCHITECTURE.md。Mutation 添加 X-CSRF-Token，需幂等的动作追加 Idempotency-Key，更新携带 version；UI 的成功提示只依据真实成功响应。未知写入结果保持原键重试，权限错误不自动重放写请求。

桌面后台以 1440px 为主要布局，至少适配 1024px；商城兼容 375px 手机。表单有 label/键盘焦点，错误信息不只依赖颜色；重要确认弹窗允许键盘操作。路由懒加载，图片懒加载，列表不一次渲染全部数据。

## 功能总表

| 模块 | 功能编号 | 功能 | 状态 |
|---|---|---|---|
| A 管理员身份与权限 | A01 | 管理员初始化、登录和退出 | ⚪ 待实现 |
| A 管理员身份与权限 | A02 | 改密、会话与权限执行 | ⚪ 待实现 |
| B 商家账户管理 | B01 | 商家列表与申请详情 | ⚪ 待实现 |
| B 商家账户管理 | B02 | 审批通过与驳回 | ⚪ 待实现 |
| B 商家账户管理 | B03 | 停用与恢复商家 | ⚪ 待实现 |
| C 商品治理 | C01 | 跨店商品搜索与查看 | ⚪ 待实现 |
| C 商品治理 | C02 | 一键下架部分商品 | ⚪ 待实现 |
| C 商品治理 | C03 | 一键下架全部商品 | ⚪ 待实现 |
| C 商品治理 | C04 | 解除商品或整店禁售 | ⚪ 待实现 |
| D 商家操作记录 | D01 | 平台查询商家日志 | ⚪ 待实现 |
| D 商家操作记录 | D02 | 管理员治理操作留痕 | ⚪ 待实现 |
| E 销售记录查询 | E01 | 全平台销售明细 | ⚪ 待实现 |
| E 销售记录查询 | E02 | 平台销售汇总与店铺维度 | ⚪ 待实现 |
| F 平台公共服务与质量 | F01 | 身份服务、网关契约与初始化 | ⚪ 待实现 |
| F 平台公共服务与质量 | F02 | 管理闭环、隔离与运维验收 | ⚪ 待实现 |

## 模块 A：管理员身份与权限

### A01 管理员初始化、登录和退出

**功能实现状态：⚪ 待实现**

**目标与接口：** `部署初始化；POST /auth/login；GET /auth/me；POST /auth/logout；GET /auth/csrf`。

**具体实现思路：** 仅登录页，无开放注册；强制改密标志引导到改密页；按权限渲染菜单并用路由守卫拦截，401 清理状态。

**联调约束：** identity 提供受部署控制的一次性管理员创建任务，幂等且默认关闭；不提交默认密码。登录强制 ADMIN audience，与 USER/SELLER 会话分域；权限由服务端分配，ADMIN_SUPER 初始拥有平台管理权限。

**示例输入与输出：** 登录 {"username":"admin_demo","password":"ExampleOnly_123"} → {"role":"ADMIN","mustChangePassword":true,"permissions":["SELLER_REVIEW"]}。

**验收条件：** 无管理员注册公网接口；消费者或商家 cookie 不能访问管理员 API。

### A02 改密、会话与权限执行

**功能实现状态：⚪ 待实现**

**目标与接口：** `POST /auth/change-password`。

**具体实现思路：** 旧密码/新密码/确认密码表单；成功后清会话要求重新登录；前端隐藏按钮仅改善体验，不代替后端授权。

**联调约束：** identity 校验旧密码后更新 BCrypt 哈希、递增 session_version 并撤销所有会话；首次改密前只允许 me/logout/change-password；admin-api 与 commerce 均校验所需权限。

**示例输入与输出：** 输入 {"oldPassword":"ExampleOnly_123","newPassword":"NewExampleOnly_456"} → {"reauthenticationRequired":true}。

**验收条件：** 旧会话即刻失效；直接调无权限 API 返回 403；敏感字段不落日志。

## 模块 B：商家账户管理

### B01 商家列表与申请详情

**功能实现状态：⚪ 待实现**

**目标与接口：** `GET /sellers；GET /sellers/{tenantId}`。

**具体实现思路：** 支持店铺名、账号、申请状态、申请日期筛选；详情显示经营资料、关联账号状态、审核历史及版本；列表不显示密码或会话。

**联调约束：** admin-api 获取 commerce tenant_profile，按需调用 identity 批量只读账号摘要；每个来源失败明确标注，不将未知当正常；权限 SELLER_REVIEW 或 SELLER_MANAGE。

**示例输入与输出：** GET /sellers?status=PENDING_APPROVAL → {"items":[{"tenantId":"401","shopName":"演示店","status":"PENDING_APPROVAL","version":1}],"total":1,"page":1,"pageSize":20}。

**验收条件：** 分页及筛选一致；未授权管理员无法读取申请联系方式。

### B02 审批通过与驳回

**功能实现状态：⚪ 待实现**

**目标与接口：** `POST /sellers/{tenantId}/review`。

**具体实现思路：** 审批弹窗展示店铺、申请版本、通过/驳回；驳回理由必填；成功刷新列表，不在本地直接切换 ACTIVE。

**联调约束：** admin-api 持 ADMIN 委托上下文调用 commerce；要求 SELLER_REVIEW；锁 tenant，身份 READY 且状态 PENDING_APPROVAL 才可审批；version+幂等键防重复；写审核历史及审计同事务。

**示例输入与输出：** 输入 {"decision":"APPROVE","version":1,"reason":"资料完整"} → {"tenantId":"401","status":"ACTIVE","version":2}；REJECT 必须给 2～200 字理由。

**验收条件：** 两个管理员同时审批只有一个生效；驳回后商家只能改资料重提。

### B03 停用与恢复商家

**功能实现状态：⚪ 待实现**

**目标与接口：** `POST /sellers/{tenantId}/suspend；POST /sellers/{tenantId}/resume`。

**具体实现思路：** 停用确认弹窗说明禁止营业/确认收款但保留历史，必填理由；恢复弹窗说明不会解除商品平台封禁；提交展示实际服务端结果。

**联调约束：** commerce 锁租户，ACTIVE→SUSPENDED 或 SUSPENDED→ACTIVE，更新版本及审计；所有商家业务实时查租户状态，停用不依赖 cookie 到期。消费者历史查询/取消、过期回收保持可用。

**示例输入与输出：** POST /sellers/401/suspend {"version":2,"reason":"演示违规停用"} → {"status":"SUSPENDED","version":3}。

**验收条件：** 停用成功后旧会话不能确认订单；历史订单和审计保留；恢复不自动上架。

## 模块 C：商品治理

### C01 跨店商品搜索与查看

**功能实现状态：⚪ 待实现**

**目标与接口：** `GET /products；GET /products/{id}`。

**具体实现思路：** 筛选店铺、关键词、商家上下架状态和平台封禁状态；明确展示所属店铺，禁止对归属含糊的商品操作。

**联调约束：** admin-api 调 commerce 平台专用查询，要求 PRODUCT_MODERATE；返回全状态商品及治理理由、版本、SKU 摘要；只读详情，不允许管理员直接篡改商家价格/库存。

**示例输入与输出：** GET /products?tenantId=401&platformStatus=BLOCKED → {"items":[{"id":"301","tenantId":"401","platformStatus":"BLOCKED"}],"total":1,"page":1,"pageSize":20}。

**验收条件：** 普通商家不能调用此跨租户查询；下架商品仍能在平台后台查到。

### C02 一键下架部分商品

**功能实现状态：⚪ 待实现**

**目标与接口：** `POST /sellers/{tenantId}/products/block`。

**具体实现思路：** 仅跨页显式选中 ID 可提交，弹窗显示店铺、数量及理由；切换店铺清空选择；最多 200 件一批，不用含糊的“当前搜索全部”。

**联调约束：** commerce 先锁 tenant 再按 ID 锁产品，核实每个 ID 属于目标租户、version 匹配；任一非法整体回滚；设置 platform_status=BLOCKED，理由和审计同事务。列表下架即时影响购买校验。

**示例输入与输出：** 输入 {"products":[{"id":"301","version":2},{"id":"302","version":1}],"reason":"演示检查"} → {"affectedCount":2,"productIds":["301","302"]}。

**验收条件：** 混入他店 ID 必须整体失败；商家重新上架无法绕过 BLOCKED；重复键结果一致。

### C03 一键下架全部商品

**功能实现状态：⚪ 待实现**

**目标与接口：** `POST /sellers/{tenantId}/catalog/block`。

**具体实现思路：** 危险操作弹窗要求输入店铺名确认；显示“整店停止销售，包含后续新商品”，不受当前分页或筛选影响；成功刷新店铺禁售标记。

**联调约束：** commerce 锁 tenant_profile 设置 sales_blocked=true、block_reason、version；无需逐条改千万商品即可同步阻止全店新交易；公开查询和结算都强制检查此开关，审计记录 whole_tenant 范围与当时商品数。

**示例输入与输出：** 输入 {"version":3,"reason":"整店检查"} → {"tenantId":"401","salesBlocked":true,"scope":"ALL_CURRENT_AND_FUTURE","version":4}。

**验收条件：** 一键覆盖非当前页商品和后来创建商品；操作提交后新订单全部拒绝。

### C04 解除商品或整店禁售

**功能实现状态：⚪ 待实现**

**目标与接口：** `POST /sellers/{tenantId}/products/unblock；POST /sellers/{tenantId}/catalog/unblock`。

**具体实现思路：** 提供独立解除入口，显示商品原销售状态，必须填理由；解除整店后仍保留商品级 BLOCKED 提示。

**联调约束：** 解除平台限制只改对应平台字段，不修改 seller_status；有效可售仍需 ACTIVE/ON_SALE/所有平台限制解除；产品解除使用 products 数组版本，整店解除使用 tenant version，同样审计和幂等。

**示例输入与输出：** POST /sellers/401/catalog/unblock {"version":4,"reason":"复核通过"} → {"salesBlocked":false,"version":5}。

**验收条件：** 解除整店不解除个别商品封禁；商家 OFF_SALE 的商品不会被平台自动上架。

## 模块 D：商家操作记录

### D01 平台查询商家日志

**功能实现状态：⚪ 待实现**

**目标与接口：** `GET /seller-audit-logs；GET /seller-audit-logs/{id}`。

**具体实现思路：** 按商家、动作、操作者、日期、订单/商品编号筛选；详情展示脱敏前后差异与 requestId，可跳转关联资源。

**联调约束：** admin-api 调 commerce 平台审计只读接口，权限 AUDIT_READ；必须限制分页和日期跨度，查阅日志自身记录管理员访问审计；不提供修改/删除日志接口。

**示例输入与输出：** GET /seller-audit-logs?tenantId=401&action=INVENTORY_ADJUST → {"items":[{"id":"901","tenantId":"401","action":"INVENTORY_ADJUST"}],"total":1,"page":1,"pageSize":20}。

**验收条件：** 筛选跨店结果有明确归属；密码/token/完整收货地址不能出现。

### D02 管理员治理操作留痕

**功能实现状态：⚪ 待实现**

**目标与接口：** `GET /admin-audit-logs`。

**具体实现思路：** 平台操作记录只读页按管理员、动作、范围和时间筛选；显示成功/失败/结果待确认，超时不能伪装成功。

**联调约束：** commerce 治理操作审计随交易提交，为事实源；admin-api 本地记录请求/权限拒绝/下游故障等访问日志，以 request_id 关联而不重复计为两次业务动作。跨库日志失败不能回滚已经生效的远程治理，需告警及恢复。

**示例输入与输出：** 输入筛选 action=CATALOG_BLOCK → {"items":[{"actorId":"1","tenantId":"401","scope":"ALL_CURRENT_AND_FUTURE","result":"SUCCESS"}],"total":1,"page":1,"pageSize":20}。

**验收条件：** 每个审批/停用/批量下架都可追溯到操作者、理由与目标范围。

## 模块 E：销售记录查询

### E01 全平台销售明细

**功能实现状态：⚪ 待实现**

**目标与接口：** `GET /sales；GET /sales/{orderId}`。

**具体实现思路：** 筛选商家、订单号、商品快照关键词、确认日期；列表明确店铺与确认人，详情查看购买时明细；不提供人工改销售金额。

**联调约束：** admin-api 调 commerce 平台销售查询，要求 SALES_READ；只查 COMPLETED 生成的记录，分页基于销售记录，详情按订单关联快照；过滤语义与商家报表一致。

**示例输入与输出：** GET /sales?tenantId=401 → {"items":[{"orderId":"701","tenantId":"401","amount":"79.80","confirmationMode":"MANUAL_DEMO"}],"total":1,"page":1,"pageSize":20}。

**验收条件：** 销售额与商家同筛选结果一致；未确认订单不混入销售。

### E02 平台销售汇总与店铺维度

**功能实现状态：⚪ 待实现**

**目标与接口：** `GET /sales/summary；GET /sales/by-seller`。

**具体实现思路：** 展示确认销售额、订单数、件数及按店汇总表；时间范围变更同时刷新，加载中禁止拿旧汇总解释新列表。

**联调约束：** commerce 用一致过滤 DTO 计算 overall 与 group by tenant；金额按订单汇总、件数独立汇总，返回 asOf；数据不断变化时不同请求可有时差，刷新使用共同 reportSnapshot 标识由服务端固定 cutoff。

**示例输入与输出：** GET /sales/summary → {"orderCount":1,"quantity":2,"salesAmount":"79.80","asOf":"2026-09-27T12:30:00Z","reportSnapshot":"1101"}。

**验收条件：** 各店汇总在同一快照下等于平台总计；标题明确为演示确认销售。

## 模块 F：平台公共服务与质量

### F01 身份服务、网关契约与初始化

**功能实现状态：⚪ 待实现**

**目标与接口：** `/internal/v1/identity/*；内网启动任务`。

**具体实现思路：** 三端登录交互遵守同一错误格式；管理员前端不持有用户/商家凭据，不提供冒充登录。

**联调约束：** identity 统一注册/登录/会话/内部委托令牌验证及 JWKS；merchant provisioning 使用任务表；初始化分类由 commerce migration 执行。网关源码归 shop-user，平台 infra 只配置三端路由，不重复开发网关。

**示例输入与输出：** 内部 introspect 合法 USER session → {"subject":"101","audience":"USER","sessionVersion":1}；错误域 → 403。

**验收条件：** 登录状态和商家审批状态没有双事实源；初始化不会反复重置密码。

### F02 管理闭环、隔离与运维验收

**功能实现状态：⚪ 待实现**

**目标与接口：** `契约测试、联合 Compose 计划`。

**具体实现思路：** Playwright 覆盖审批、拒绝、停用、部分/全部下架、解除、查日志、查销售；双管理员并发验证版本冲突提示。

**联调约束：** 测试治理与下单竞态、越权、CSRF、会话失效、下游超时后的原键重试；数据库迁移使用 Flyway，基础设施与密钥仅环境注入，监测审批失败率及 commerce 可用性。

**示例输入与输出：** 审批 T1 → 上架 → U1 下单 → T1 确认 → 平台查销售；禁售 T1 后再次下单应失败。

**验收条件：** 闭环及故障测试全部通过才能标 ✔；本次无程序、不执行虚构运行验收。

## 页面、状态及权限映射

Pinia 划分：authStore、sellerReviewStore、productSelectionStore、salesReportStore。

| 模块 | 路由规划 | 状态和权限边界 |
|---|---|---|
| A | /login、/change-password | ADMIN 身份；首次改密限制 |
| B | /sellers、/sellers/:id | SELLER_REVIEW / SELLER_MANAGE |
| C | /products、/sellers/:id/products | PRODUCT_MODERATE |
| D | /seller-audit-logs、/admin-audit-logs | AUDIT_READ |
| E | /sales | SALES_READ |
| F | /403、/404 | ADMIN 与服务端权限 |

## 实施步骤及交付判定

1. 确认共同接口合同与版本锁定，建立工程骨架、认证及统一错误处理；本次不生成该骨架。
2. 按 A→B→C 的业务依赖实现，再补齐 D/E 的查询和记录；F 的安全约束从第一模块贯穿实施，不留到最后才加租户过滤。
3. 每个编号至少有正常路径、非法参数、无权限/非所属、状态冲突、重复提交（若为写操作）验收记录。前端组件测试与后端业务测试各自独立。
4. 使用共同文档的联合验收场景完成三仓库联调。示例响应由 mock 驱动界面时必须注明 mock，不视为后端完成。
5. 完成后只更新真正通过的功能状态为 ✔，附实现提交 SHA、测试报告位置和日期；其余继续 ⚪，不因为“文档已完成”而标程序已实现。

## 非本期范围

真实支付/退款、物流、优惠券、商家员工系统、短信邮箱验证服务、复杂多币种、跨店原子结算、搜索引擎和数据仓库不在本期；未设计的增量需求需新增功能编号及兼容合同后实现。管理员销售导出不是本次必需项，商家 XLSX 导出为本期必需项。
