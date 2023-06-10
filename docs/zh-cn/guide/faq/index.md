# FAQ

[[toc]]

## Vue warn: Failed setting prop

```
[Vue warn]: Failed setting prop "prefix" on <component-stub>: value foo is invalid.
TypeError: Cannot set property prefix of #<Element> which has only a getter
```

这个警告，是在你用 `shallowMount` 或者 `stubs` 时，使用了与 [`Element`](https://developer.mozilla.org/en-US/docs/Web/API/Element) 共享的属性名的时候出现的。

常见的与 `Element` 共享的属性有：
* `attributes`
* `children`
* `prefix`

见： https://developer.mozilla.org/en-US/docs/Web/API/Element

**可行的方案**

1. 使用 `mount` 替代 `shallowMount`，不以模拟对象进行渲染
2. 通过 mock `console.warn` 来忽略这个警告
3. 从命名 prop 是它与 `Element` 的属性名不冲突
