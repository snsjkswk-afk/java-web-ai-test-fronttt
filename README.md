# JavaWeb AI Test Frontend（tlias 前端）

学生信息管理系统的前端，基于 Vue 3 + Vite + Element Plus 构建，通过 Axios 调用 [java-web-ai-test-back](https://github.com/snsjkswk-afk/java-web-ai-test-back) 提供的 REST 接口。

- 登录态基于 JWT，token 存于 `localStorage`，请求拦截器自动注入请求头
- Element Plus 中文语言包 + 全量图标注册，表格/表单/弹窗开箱可用
- 部门、员工、班级、学生、违纪记录、操作日志六个业务页面 + 仪表盘图表（ECharts）
- 开发环境通过 Vite 代理解决跨域，生产构建输出纯静态文件，可交给 Nginx 托管

## 技术栈

| 分类 | 技术 | 版本 |
|------|------|------|
| 框架 | Vue | 3.4 |
| 构建 | Vite | 5.4 |
| 路由 | Vue Router | 4.3 |
| 状态管理 | Pinia | 2.1 |
| UI 组件 | Element Plus / @element-plus/icons-vue | 2.7 / 2.3 |
| 请求 | Axios | 1.7 |
| 图表 | ECharts | 5.5 |
| 运行时 | Node.js | 18+ |

## 目录结构

```
frontend
├── index.html
├── vite.config.js          # 端口 5173，/api 代理到后端 8080
├── package.json
└── src
    ├── main.js             # 应用入口：Pinia、Router、Element Plus、图标注册
    ├── App.vue
    ├── router/index.js     # 路由表（登录页 + Layout 下的业务页面）
    ├── layout/Layout.vue   # 侧边栏 + 顶栏 + 内容区
    ├── utils/request.js    # Axios 实例、token 注入、统一响应处理
    └── views
        ├── Login.vue       # 登录
        ├── Dashboard.vue   # 数据看板
        ├── Dept.vue        # 部门管理
        ├── Emp.vue         # 员工管理（含工作经历子表单）
        ├── Clazz.vue       # 班级管理
        ├── Student.vue     # 学生管理
        ├── Violation.vue   # 违纪记录
        └── Log.vue         # 操作日志
```

## 页面与路由

| 路径 | 页面 | 对应后端接口 |
|------|------|--------------|
| `/login` | 登录 | `POST /login` |
| `/dashboard` | 仪表盘（职位/性别/学历/班级人数图表） | `/report/*` |
| `/dept` | 部门管理 | `/depts` |
| `/emp` | 员工管理 | `/emps` |
| `/clazz` | 班级管理 | `/clazzs` |
| `/student` | 学生管理 | `/students` |
| `/violation` | 违纪记录 | `/students/{id}/violations` |
| `/log` | 操作日志 | `/log/page` |

未登录访问业务页面会被重定向到 `/login`。

## 快速开始

### 1. 环境要求

- Node.js 18 及以上（Vite 5 要求）
- 后端服务已在 `http://localhost:8080` 启动

### 2. 安装依赖

```bash
git clone https://github.com/snsjkswk-afk/java-web-ai-test-fronttt.git
cd frontend
npm install
```

### 3. 启动开发服务器

```bash
npm run dev
```

浏览器访问 `http://localhost:5173`。

### 4. 后端联调

`vite.config.js` 已配置代理，前端请求统一以 `/api` 开头，实际转发到后端并去掉 `/api` 前缀：

```js
server: {
  port: 5173,
  proxy: {
    '/api': {
      target: 'http://localhost:8080',
      changeOrigin: true,
      rewrite: (path) => path.replace(/^\/api/, '')
    }
  }
}
```

即前端 `POST /api/login` → 后端 `POST http://localhost:8080/login`。若后端换了地址或端口，只改这里的 `target`。

## 请求与鉴权

统一实例位于 `src/utils/request.js`：

```js
const request = axios.create({ baseURL: '/api', timeout: 10000 })
```

- **请求拦截器**：自动从 `localStorage.token` 取出 token，写入请求头 `token`
- **响应拦截器**：`code === 1` 视为成功并返回 `res`；否则弹出 `ElMessage.error(msg)` 并 reject
- **401 处理**：清除本地 token 与用户信息，跳转登录页

接口返回结构：

```json
{ "code": 1, "msg": "success", "data": {} }
```

## 构建与部署

```bash
npm run build      # 产物输出到 dist/
npm run preview    # 本地预览构建结果
```

用 Nginx 托管 `dist/` 时，需要同时配置前端路由回退与接口代理：

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    # history 路由回退
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 接口转发到后端
    location /api/ {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 开发约定

- 提交信息遵循 Conventional Commits：`feat` / `fix` / `style` / `chore` / `docs`，格式 `类型(范围): 简述`
- `node_modules/`、`dist/`、`dist.zip` 不入库，已在 `.gitignore` 中忽略
- 新增页面：在 `src/views` 下建组件 → 在 `src/router/index.js` 注册路由 → 在 `src/layout/Layout.vue` 侧边栏加菜单
- 接口统一走 `src/utils/request.js`，不要在组件里直接 `axios.get`，以免绕过 token 注入与错误提示

## 相关仓库

- 后端：https://github.com/snsjkswk-afk/java-web-ai-test-back （Spring Boot 3 + MyBatis + MySQL）

## 说明

本项目为课程作业，用于演示前后端分离架构下的页面组织、路由守卫、请求封装与接口联调，未做移动端适配与生产级性能优化。
