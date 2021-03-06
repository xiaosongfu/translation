---
title: "Cargo 的未来几年"
date: 2019-02-13T15:39:25+08:00
draft: false
tags: ["Rust", "Cargo", "tag3"]
categories: ["Rust"]
---

> 原文链接：[Cargo's Next Few Years](https://www.ncameron.org/blog/cargos-next-few-years/)

Cargo 团队一直在思考和讨论 Cargo 的长期计划。在这篇文章中，我将谈谈我们希望 Cargo 在下一版本的时候会是什么样子（假设有另一版本，并且它发生在大约三年内，两者都没有得到证实）。今年将有更多具体计划的另一篇文章，包括某种路线图。

通过消除构建，运行和分发 Rust 代码的障碍，Cargo 使“系统”编程可以访问。这至少就是Cargo 对我的意义。但这有点低估了它;Cargo 是一个项目的力量倍增器：它降低了贡献的障碍，这意味着更多人可以更容易地参与其中，并且可以轻松地以一致的方式试用软件。例如，如果你已经听说过这个很酷的项目 ripgrep（就像 grep，但更快更好）并且你想尝试一下，那么你只需要运行 `cargo install ripgrep` 就可以了。如果你觉得它很棒并想要贡献，那么在 `git clone` 之后，你所要知道的就是运行 `cargo build` 或 `cargo test` 来开始。

请注意，我没有谈了 **构建系统** 或 **包管理**。这些是 Cargo 所做的事情，但它们不是最重要的问题。

## Cargo 在 2022年

Cargo 是一个很好的工具。在接下来的几年里，我不想优先考虑做更多的事情或做不同的事情。相反，我想专注于为更多人做同样的事情。我希望我们将 Cargo 魔力带给那些在一个更大的项目中使用 Rust 的人，这个项目有自己的构建系统，或者交叉编译（比如WASM）。我希望 Cargo 在项目有资源需要提供或处理的地方工作，等等。

我不会假装这是未来三年的完整愿景。我认为这是一个起点，明年我们将努力使其更加具体，特别是要弄清楚“为这些新场景带来 Cargo 魔力”意味着什么。

## 版本周期

我们刚刚发布了 2018 edition（在8周前），下一版可能会在大约三年后发布。2018 edition 非常棒，但它很辛苦，并且有很多变化。有些人一直呼吁减少变革或减缓变革步伐。

我认为一个好的节奏是专注于实验，并在第一年偿还技术和组织债务，专注于第二年的设计和实施，并专注于整合，“整理事物”，以及贬低/删除第三年。

同样值得注意的是，Cargo 对于该版本并没有太大的改变，所以我认为这里的开发和实验比其他领域更有胃口。

## 我们如何到达那里

我们需要关注我们的工作。我认为我们应该一次关注一组用户，并使用一组小的协调功能。我们应该确保它们与货物的其余部分很好地整合，并且最终结果不会是一个臃肿的混乱。如上所述，我们还应该考虑时间安排，特别是在编辑周期方面。

我建议我们关注以下领域：

* 2019年：交叉编译，包括wasm和embeddded，
* 2020：将Cargo嵌入构建系统和IDE中，
* 2021：定制最终用户工作流程。

让我们深入了解每个人的意思，以及为什么我选择了这个时机

## 其他的东西

值得一提的是，还有很多其他应该为 Cargo 计划的东西，包括：为 Rust 项目服务其他目标的功能（例如，减少编译时间），支付技术债务，抛光现有功能，稳定性，并改进文档。


