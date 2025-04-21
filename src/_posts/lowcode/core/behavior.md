---
title: 低代码核心实现 - 组件间数据和行为交互
date: 2025-04-17 10:44:02
category: LowCode
tags: [LowCode]
cover:
---

每一个组件可以对外提供：

1. 抛出可联动的数据
2. 抛出可调用的方法
3. 提供可监听的事件

### 抛出可联动的数据

每个组件可以向外抛出数据，应用会收集维护在全局存储，该数据应是可响应式的，在应用顶层通过 Provider 向下传递，每一个组件都可以读取到。

组件抛出的数据，在编辑器中拖动配置组件时，为了能够被其它组件使用，需要在该组件中预先定义好待抛出的数据。

```typescript
export default defineComponent<Options>({
  declarations: {
    // 此组件可联动的数据
    linkState: {
      // 定义此组件会向外抛出”已预约人数“的数据
      numFollowed: {
        label: '已预约人数',
        type: 'number',
        default: 0,
      },
    },
  },
});
```

组件实现中借助统一提供的函数抛出数据：

```typescript
const num = await getFollowedNum(); // 示例从接口获取

// 抛出函数由全局下发，内部实现时，需关联当前组件的唯一 ID 将其挂载到全局存储
syncState({
  numFollowed: num,
});
```

而其它组件可通过以下方式使用数据：

1. 自己的配置项，直接绑定来自于某组件下的某数据

2. 配置联动目标组件的某项数据，当其”大于、小于、等于“目标值时，联动切换指定的状态

### 抛出可调用的方法

组件中定义可调用的方法名，及配置方法时的表单配置。

```typescript
export default defineComponent<Options> ({
  declarations: {
    // 可被其它组件调用的方法
    methods: [
      {
        label: '刷新列表',
        name: 'refresh',
        form: {
          label: '刷新第几页'
          placeholder: '请输入刷新第几页，若缺省默认刷新当前页'
        }
      }
    ]
  }
})
```

```typescript
const refresh = () => {};

// 示例 Vue 的实现
defineExpose({
  refresh,
});
```

### 提供可监听的事件

```typescript
export default defineComponent<Options>({
  declarations: {
    // 可被其它组件监听的事件
    events: [
      {
        label: '加载完成时',
        name: 'loaded',
      },
    ],
  },
});
```
