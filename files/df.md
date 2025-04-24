说明：
vue3实现router路由
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a81325dcf09d4bb49d1f4e04fe7c20c2.png#pic_center)

step1:项目结构

```bash
src/
├── views/
│   ├── Home.vue
│   └── User.vue
├── router/
│   └── index.js
├── App.vue
└── main.js
```

step2:左边路由列表C:\Users\wangrusheng\PycharmProjects\untitled3\src\App.vue

```typescript
<!-- App.vue -->
<template>
  <div class="container">
    <!-- 左侧导航 -->
    <div class="sidebar">
      <h2>路由列表</h2>
      <nav>
        <router-link to="/home" active-class="active-link">首页</router-link>
        <router-link to="/user" active-class="active-link">用户中心</router-link>
      </nav>
    </div>

    <!-- 右侧内容 -->
    <div class="content">
      <router-view></router-view>
    </div>
  </div>
</template>

<style>
.container {
  display: flex;
  height: 100vh;
}

.sidebar {
  width: 200px;
  background-color: #f0f0f0;
  padding: 20px;
  border-right: 1px solid #ddd;
}

.sidebar nav {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.sidebar a {
  padding: 10px;
  text-decoration: none;
  color: #333;
  border-radius: 4px;
  transition: all 0.3s;
}

.sidebar a:hover {
  background-color: #e0e0e0;
}

.active-link {
  background-color: #4CAF50;
  color: white !important;
}

.content {
  flex: 1;
  padding: 20px;
}
</style>
```

step3:配置路由 C:\Users\wangrusheng\PycharmProjects\untitled3\src\router\index.js

```typescript
// src/router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import User from '../views/User.vue'

const routes = [
  { path: '/', redirect: '/home' },
  { path: '/home', name: 'Home', component: Home },
  { path: '/user', name: 'User', component: User }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

step4:主界面引入配置js C:\Users\wangrusheng\PycharmProjects\untitled3\src\main.js

```typescript
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'
import router from './router'


createApp(App).use(router).mount('#app')


```

step5:home C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\Home.vue

```typescript
<!-- src/views/Home.vue -->
<template>
  <div class="content">
    <h1>Home Page</h1>
    <p>Hello World from Home!</p>
  </div>
</template>

<script>
export default {
  name: 'HomeView'
}
</script>
```

step6:user C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\User.vue

```typescript
<!-- src/views/User.vue -->
<template>
  <div class="content">
    <h1>User Page</h1>
    <p>Hello World from User!</p>
  </div>
</template>

<script>
export default {
  name: 'UserView'
}
</script>
```

end