# 关于此文档

本文档从引用参考和概念两个方面全面的解释了 Node.js API。每个章节描述了一个模块或高级概念。

一般情况下，属性、方法参数，及事件都会列在主标题下的列表中。

每个 `.html` 文档都有对应的 `.json` ，它们包含相同的结构化内容。这些东西目前还是实验性的，主要为各种集成开发环境（IDE）和开发工具提供便利。

每个 `.html` 和 `.json` 文件都和 `doc/api/` 目录下的 `.markdown` 文件相对应。这些文档使用 `tools/doc/generate.js` 程序生成。 HTML 模板位于 `doc/template.html`。

## 稳定性标志

在文档中，你会看到每个章节的稳定性标志。Node.js API 还在改进中，成熟的部分会比其他章节值得信赖。经过大量验证和依赖的 API 是一般是不会变的。其他新增的，试验性的，或被证明具有危险性的部分正重新设计。

稳定性标志包括以下内容:

```
稳定性（Stability）: 0 - 抛弃
这部分内容有问题，并已计划改变。不要使用这些内容，可能会引起警告。不要想什么向后兼容性了。
```

```
稳定性（Stability）: 1 - 试验
这部分内容最近刚引进，将来的版本可能会改变也可能会被移除。你可以试试并提供反馈。如果你用到的部分对你来说非常重要，可以告诉 node 的核心团队。
```

```
稳定性（Stability）: 2 - 不稳定
这部分 API 正在调整中，还没在实际工作测试中达到满意的程度。如果合理的话会保证向后兼容性。
```

```
稳定性（Stability）: 3 - 稳定
这部分 API 验证过基本能令人满意，但是清理底层代码时可能会引起小的改变。保证向后的兼容性。
```

```
稳定性（Stability）: 4 - API 冻结
这部分的 API 已经在产品中广泛试验，不太可能被改变。
```

```
稳定性（Stability）: 5 - 锁定
除非发现了严重 bug，否则这部分代码永远不会改变。请不要对这部分内容提出更改建议，否则会被拒绝。

```

## JSON 输出

    稳定性（Stability）: 1 - 试验

每个通过 markdown 生成的 HTML 文件都有相应的 JSON 文件。

这个特性从 v0.6.12 开始,是试验性功能。
