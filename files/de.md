说明：
用react实现router路由
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/186041362d7443d1b88a95bb4e2a1e47.png#pic_center)

step0:项目结构图：

```bash
my-react-app/
├── public/                  # 静态资源
│   ├── favicon.ico
│   └── robots.txt
├── src/
│   ├── assets/             # 静态资源
│   ├── pages/              # 页面组件
│   │   ├── Home.jsx           # 首页模块
│   │   └── User.jsx         # 用户模块
│   ├── App.jsx
│   └── main.jsx
```

step1:C:\Users\wangrusheng\PycharmProjects\untitled16\src\main.jsx

```typescript
// src/main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { BrowserRouter } from 'react-router-dom'
import App from './App.jsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
)
```

step2:C:\Users\wangrusheng\PycharmProjects\untitled16\src\App.jsx

```typescript
// App.jsx
import { NavLink, Routes, Route, Navigate } from 'react-router-dom'
import Home from './pages/Home'
import User from './pages/User'
import './App.css' // 新增 CSS 文件

export default function App() {
  return (
    <div className="app-container">
      {/* 左侧导航 */}
      <div className="sidebar">
        <h2>路由列表</h2>
        <nav>
          <NavLink
            to="/home"
            className={({ isActive }) =>
              `nav-link ${isActive ? 'active' : ''}`
            }
          >
            首页
          </NavLink>
          <NavLink
            to="/user"
            className={({ isActive }) =>
              `nav-link ${isActive ? 'active' : ''}`
            }
          >
            用户中心
          </NavLink>
        </nav>
      </div>

      {/* 右侧内容 */}
      <div className="content">
        <Routes>
          <Route path="/" element={<Navigate to="/home" />} />
          <Route path="/home" element={<Home />} />
          <Route path="/user" element={<User />} />
        </Routes>
      </div>
    </div>
  )
}
```

step3:C:\Users\wangrusheng\PycharmProjects\untitled16\src\App.css

```css
/* App.css */
.app-container {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  width: 200px;
  background-color: #f5f5f5;
  padding: 20px;
  border-right: 1px solid #ddd;
}

.sidebar h2 {
  margin-bottom: 1rem;
  color: #333;
}

.nav-link {
  display: block;
  padding: 10px;
  text-decoration: none;
  color: #333;
  border-radius: 4px;
  margin-bottom: 8px;
  transition: all 0.3s ease;
}

/* 鼠标悬停效果 */
.nav-link:hover {
  background-color: #e0e0e0;
}

/* 选中状态 */
.nav-link.active {
  background-color: #4CAF50;
  color: white !important;
}

.content {
  flex: 1;
  padding: 20px;
  background-color: white;
}
```

step4:C:\Users\wangrusheng\PycharmProjects\untitled16\src\pages\User.jsx

```typescript
// src/pages/User.jsx
export default function User() {
  return (
    <div>
      <h2>用户中心</h2>
      <p>Hello World from User!</p>
    </div>
  )
}
```

step5:C:\Users\wangrusheng\PycharmProjects\untitled16\src\pages\Home.jsx

```typescript
// src/pages/Home.jsx
export default function Home() {
  return (
    <div>
      <h2>首页内容</h2>
      <p>Hello World from Home!</p>
    </div>
  )
}
```

end