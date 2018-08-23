# Vue 常用组件使用方法

## 收藏组件使用

1. 首先, 需要引用收藏组件
```vue.js
import BookMark from '@/components/BookMark.vue'
export default {
  components: {
  vbookmark: BookMark
   },
}
```

2. 使用收藏组件

```vue.js
<vbookmark :status="bmark.collection" :postId="bmark.id" :userId="userInfo.userId" @callback="refreshBookMark"/>
```

3. 实现callback函数

```vue.js
methods: {
    refreshBookMark() {
      // 点完收藏后可以做一些事情. 如需要更新修改后的数据
    },
}
```
