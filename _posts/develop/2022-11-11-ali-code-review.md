---
layout: post
title: 阿里技术:Code Review
date: 2022-11-11 16:12:15
categories: 总结  
tags:  工作经验 
excerpt: 代码 review 规范是一个团队能力的体现
---

阿里技术微信公众号：[七个建议让 Code Review 高效又高质](https://mp.weixin.qq.com/s/AejIOMNxsI_Ru5G0W8AeTg)


# 为什么要做代码评审

代码评审最本质的作用不是问题发现。除了代码评审，我们有更多更好的手段来发现问题。代码评审的作用更多是**关于社会学的，是一种长期行为和组织文化**。

社会学？上了很高的层次了。

## 1. 编码者视角：良性的社交压力


一旦想到你的代码将会发出去给你的同事做 Review，有没有为刚才的这种想法产生一丝丝压力？这种压力是良性的，它能给你带来一种即时的反馈，阻止你去选择那些短期收益、长期损失的 “投机” 行为。如果没有代码评审这个环节，或许你就会真的 “为所欲为” 了，其实最终还是要为这种取巧行为买单。


## 2. 维护者视角：代码可读性的保证

这段代码在编码完成之后，立即经历了可读性的检验。更理想地，如果组织已经有了编码规范和设计规范，还能确保这段代码遵循了这些规范。如果 CR 过程中发现这段代码没有遵循规范，那更是好事，它指向了 CR 的另外一个关键价值：知识传播。


## 3. CR 带来了知识传播和设计共识


抽象的概念如果不落到具体的事情上，就很难形成共识。有人或许知道海洋系的 “判例”，这是在法律层面形成共识的一种非常好的方法。代码评审，其实也是在形成判例：哪一类设计是合理的，哪一类设计是不合理的。通过针对具体的问题进行分析，团队就会逐渐形成设计共识，在过程中，对这些共识不那么熟悉的新同学，也可以慢慢融入。

## 4. 检验逻辑正确性

1、保证代码逻辑正确，是设计者的责任。

2、发现逻辑错误的其他方法。

3、编写自动化单元测试。

4、使用代码静态检查工具。

## 实际操作建议

1、小批量——每次 Review 的代码量要少。

2、多批次——Review 要频繁发生。

3、找对人——合适的 Reviewer。

4、快速响应。

5、使用现代工具。

6、考虑结对编程。

7、综合在线 Review 和线下 Review。

---
参考资料：

[# 七个建议让 Code Review 高效又高质](https://mp.weixin.qq.com/s/AejIOMNxsI_Ru5G0W8AeTg)
