# 03 注意力工程平移：7 种机制 → 电商 AI 指令设计

Superpowers 的 7 种注意力捕获机制是纯方法论——不依赖软件开发上下文，可以直接平移到电商 AI 的指令设计。

## 7 种机制的电商适配

```
  ┌──────────────────────────────────────────────────────────┐
  │  机制           开发场景              电商场景            │
  │  ────           ────────            ────────            │
  │  XML 标签       HARD-GATE           COMPLIANCE-GATE      │
  │  铁律           NO CODE WITHOUT TEST NO REFUND W/O CHECK │
  │  宣布仪式       "I'm using [skill]"  "[风控] 处理订单"   │
  │  流程图         dot 图              电商运营流程图        │
  │  清单           TDD 验证清单        上架/退款 SOP        │
  │  Good/Bad       代码正反例          业务决策正反例        │
  │  红旗表         "I'll test after"   "This looks normal"  │
  └──────────────────────────────────────────────────────────┘
```

## XML 标签的电商适配

```
  ┌──────────────────────────────────────────────────────────┐
  │  电商 XML 标签设计：                                     │
  │                                                          │
  │  <COMPLIANCE-GATE>                                       │
  │  在确认商品符合所有法规要求之前，                         │
  │  不得批准商品上架。这适用于所有品类，                     │
  │  无论商品看起来多么合规。                                 │
  │  </COMPLIANCE-GATE>                                      │
  │  → 对应 HARD-GATE，用于商品上架、营销文案等合规场景       │
  │                                                          │
  │  <RISK-GATE>                                             │
  │  在完成风险评估之前，不得批准任何订单。                   │
  │  这适用于所有订单金额，无论金额大小。                     │
  │  </RISK-GATE>                                            │
  │  → 对应 HARD-GATE，用于风控、退款审批场景                 │
  │                                                          │
  │  <APPROVED-EXAMPLE> ... </APPROVED-EXAMPLE>               │
  │  <REJECTED-EXAMPLE> ... </REJECTED-EXAMPLE>               │
  │  → 对应 Good/Bad，用于业务决策对比学习                   │
  │                                                          │
  │  信号强度排序：                                          │
  │  RISK-GATE > COMPLIANCE-GATE > APPROVED/REJECTED          │
  │  → 风控 > 合规 > 示例                                   │
  │  → 电商中"防止资金损失" > "防止合规违规" > "学习"       │
  └──────────────────────────────────────────────────────────┘
```

## 铁律的电商适配

```
  ┌──────────────────────────────────────────────────────────┐
  │  电商铁律设计：                                          │
  │                                                          │
  │  风控铁律：                                              │
  │  ```                                                     │
  │  NO ORDER APPROVAL WITHOUT RISK VERIFICATION              │
  │  ```                                                     │
  │  审批订单前没做风控验证？拒绝审批。                       │
  │  无例外：                                                │
  │  - 不能因为"看起来正常"就跳过风控                       │
  │  - 不能因为"金额小"就跳过风控                           │
  │  - 不能因为"老客户"就跳过风控                           │
  │  - 不能因为"赶时间"就跳过风控                           │
  │                                                          │
  │  退款铁律：                                              │
  │  ```                                                     │
  │  NO REFUND WITHOUT EVIDENCE VERIFICATION                   │
  │  ```                                                     │
  │  退款前没验证证据？暂停退款。                             │
  │  无例外：                                                │
  │  - 不能因为"用户很急"就跳过验证                         │
  │  - 不能因为"金额小"就跳过验证                           │
  │  - 不能因为"客服压力大"就跳过验证                       │
  │  - 不能因为"看起来是真的"就跳过验证                     │
  │                                                          │
  │  电商铁律 vs. TDD 铁律的关键差异：                       │
  │  TDD: "写代码前没测试？删除代码，重新开始。"              │
  │  → 后果是可逆的（代码可以重写）                          │
  │                                                          │
  │  电商: "审批前没风控？拒绝审批。"                         │
  │  → 后果可能是不可逆的（拒绝真实订单 = 永久损失客户）     │
  │  → 电商铁律需要"申诉机制"：                             │
  │    AI 严格执行铁律 → 如果被误杀 → 人工申诉通道           │
  │  → TDD 不需要申诉——因为删除代码没有损失                 │
  │  → 电商需要——因为拒绝订单有真实商业损失                  │
  └──────────────────────────────────────────────────────────┘
```

## 红旗表的电商适配

```
  ┌──────────────────────────────────────────────────────────┐
  │  风控 AI 的红旗表：                                      │
  │  | Thought | Reality |                                   │
  │  | "This order looks normal"                              │
  │  |   Normal-looking orders can be fraud. Verify. |        │
  │  | "The user has a good history"                          │
  │  |   Account takeover changes everything. Re-verify. |    │
  │  | "It's just a small amount"                             │
  │  |   Small orders test your fraud detection. Flag. |      │
  │  | "We'll catch it later"                                │
  │  |   Later = after the money is gone. Block now. |        │
  │  | "The system already approved it"                       │
  │  |   Automated systems miss sophisticated fraud. |        │
  │                                                          │
  │  退款 AI 的红旗表：                                      │
  │  | Thought | Reality |                                   │
  │  | "The user provided a photo"                            │
  │  |   Photos can be from other orders. Cross-check. |      │
  │  | "They've never refunded before"                        │
  │  |   First-time fraud is the hardest to detect. |         │
  │  | "Just refund it, it's cheaper"                         │
  │  |   Cheap refunds attract fraud rings. Verify. |         │
  │  | "The product is cheap anyway"                          │
  │  |   Cheap products = high volume fraud potential. |      │
  │                                                          │
  │  选品 AI 的红旗表：                                      │
  │  | Thought | Reality |                                   │
  │  | "This category is trending"                            │
  │  |   Trends attract competition. Check margins. |         │
  │  | "The supplier is reliable"                             │
  │  |   Reliability can change. Re-verify. |                  │
  │  | "We can price it low and win on volume"                │
  │  |   Price wars destroy margins. Check sustainability. |  │
  │  | "Our competitors are selling it"                       │
  │  |   Following = no differentiation. Check unique value. ||  │
  └──────────────────────────────────────────────────────────┘
```

## Good/Bad 对比在电商中的价值

```
  ┌──────────────────────────────────────────────────────────┐
  │  退款审核的 Good/Bad 对比：                              │
  │                                                          │
  │  <APPROVED-EXAMPLE>                                      │
  │  退款申请：订单 #12345                                   │
  │  原因：商品与描述不符                                    │
  │  证据：3 张实物照片 + 1 张商品页截图                     │
  │  验证：订单已签收、物流已完结、用户首次退款              │
  │  决策：批准退款                                          │
  │  依据：证据充分、原因匹配退货政策、用户历史良好          │
  │  </APPROVED-EXAMPLE>                                     │
  │                                                          │
  │  <REJECTED-EXAMPLE>                                      │
  │  退款申请：订单 #67890                                   │
  │  原因："东西不好"                                       │
  │  证据：无                                                │
  │  验证：订单未签收、物流在途                              │
  │  决策：拒绝退款，要求补充证据                            │
  │  依据：原因模糊、无证据、订单状态不匹配                  │
  │  </REJECTED-EXAMPLE>                                      │
  │                                                          │
  │  对比学习的效果：                                        │
  │  → AI 学会了：有效退款 vs. 无效退款                      │
  │  → 关键差异：证据充分性 + 订单状态匹配 + 退货原因具体   │
  │  → 比纯规则"要求提供证据"更有效地锚定正确行为          │
  └──────────────────────────────────────────────────────────┘
```
