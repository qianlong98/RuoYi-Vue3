# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RuoYi-Vue3 is the frontend of a Chinese admin management system built with Vue 3 + Element Plus + Vite. It pairs with a Spring Boot backend (separate repository). The system features RBAC permission control, dynamic routing, dictionary management, and code generation tools.

## Build Commands

```bash
# Install dependencies (use yarn with Chinese mirror)
yarn --registry=https://registry.npmmirror.com

# Development server (runs on port 80, proxies to localhost:8080)
yarn dev

# Production build
yarn build:prod

# Staging build
yarn build:stage
```

## Architecture

### Backend API Proxy
- Development: `/dev-api` proxies to `http://localhost:8080` (configured in `vite.config.js`)
- Production: `/prod-api` is the base API path
- Ensure the backend is running before starting frontend development

### Dynamic Routing & Permission System
Routes are loaded dynamically from backend based on user permissions:
1. `src/permission.js` - Route guards check token, fetch user info, generate routes
2. `src/store/modules/permission.js` - Fetches router data from backend via `getRouters()`, transforms component strings to actual components
3. Component strings like `'system/user/index'` map to `src/views/system/user/index.vue`

Permission control mechanisms:
- `$auth.hasPermi()` / `$auth.hasRole()` - Plugin methods for programmatic checks
- `v-hasPermi` / `v-hasRole` - Directives that remove elements lacking permission
- Route `meta.permissions` / `meta.roles` - Access control for dynamic routes

### State Management (Pinia)
Key stores in `src/store/modules/`:
- `user.js` - Login/logout, user info, roles, permissions
- `permission.js` - Dynamic route generation and storage
- `app.js` - App state (sidebar, device type)
- `dict.js` - Dictionary data caching
- `tagsView.js` - Tab view management
- `settings.js` - Theme/layout settings

### Global Utilities (mounted on app.config.globalProperties)
Available in all components via `this`:
- `useDict(dictType)` - Fetch dictionary data, returns reactive refs
- `parseTime(time, pattern)` - Date formatting
- `resetForm(refName)` - Form reset
- `handleTree(data, id, parentId, children)` - Build tree structure from flat data
- `addDateRange(params, dateRange, propName)` - Add date range to search params
- `selectDictLabel(datas, value)` - Get dictionary label for value
- `download(url, params, filename)` - File download with loading indicator

### Global Components
Registered in `src/main.js`:
- `<Pagination>` - Table pagination with page size options
- `<DictTag>` - Display dictionary value with styled tag
- `<FileUpload>` / `<ImageUpload>` - Upload components
- `<RightToolbar>` - Table toolbar with refresh/column toggle
- `<Editor>` - Rich text editor (Quill-based)

### API Layer
- `src/utils/request.js` - Axios wrapper with interceptors:
  - Auto-adds Bearer token to requests
  - Handles 401 (session expired), 500, 601 errors
  - Prevents duplicate form submissions
- API files in `src/api/` mirror backend controller structure

### Layout System
`src/layout/index.vue` is the main layout with:
- Sidebar navigation (`src/layout/components/Sidebar/`)
- TagsView for tab management
- Navbar with user dropdown, screen full, size select
- Settings panel for theme configuration

### Style System
- `src/assets/styles/` - SCSS styles including element-ui overrides, sidebar, buttons
- Element Plus dark mode supported via `element-plus/theme-chalk/dark/css-vars.css`
- `src/assets/icons/svg/` - Custom SVG icons loaded via vite-plugin-svg-icons

### Vite Plugins (vite/plugins/)
- `auto-import.js` - Auto-import Vue APIs (ref, computed, etc.)
- `svg-icon.js` - SVG icon registration
- `setup-extend.js` - Support `<script setup name="xxx">` syntax
- `compression.js` - Build compression (gzip/brotli)

## Route Meta Configuration

```javascript
meta: {
  title: '页面标题',       // Sidebar/breadcrumb display name
  icon: 'svg-name',        // Icon from src/assets/icons/svg/
  noCache: true,           // Skip keep-alive caching
  hidden: true,            // Hide from sidebar
  breadcrumb: false,       // Hide from breadcrumb
  activeMenu: '/path',     // Highlight sidebar item when on this route
  permissions: ['a:b:c'],  // Permission codes required
  roles: ['admin']         // Roles required
}
```

## Key Patterns

### Adding a new view with permission
1. Create Vue file in `src/views/`
2. Add API methods in corresponding `src/api/` file
3. Backend: add menu entry with component path (e.g., `system/user/index`)
4. Use `v-hasPermi` on buttons: `<el-button v-hasPermi="['system:user:add']">`

### Dictionary usage in components
```javascript
// In setup
const { status_type, user_gender } = useDict('status_type', 'user_gender')

// In template
<el-option v-for="dict in status_type" :key="dict.value" :label="dict.label" :value="dict.value" />
<dict-tag :options="status_type" :value="row.status" />
```

### Tree data handling
Backend returns flat list with `id` and `parentId`. Use `handleTree()` to convert:
```javascript
const treeData = handleTree(deptList, 'deptId', 'parentId', 'children')
```