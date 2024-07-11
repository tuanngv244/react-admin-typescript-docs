# Cấu trúc dự án CMS Admin - React & Typescript

Dự án CMS Admin - React & Typescript được xây dựng dựa trên các công nghệ React, TypeScript, Material UI, Apexcharts và nhiều thư viện liên quan khác.

## I. Khởi tạo dự án

- Cài đặt NodeJS.
- Sử dụng lệnh `npm create vite@latest` hoặc `yarn create vite` để khởi tạo dự án React với [Vite](https://vitejs.dev/) (Khuyến khích dùng Yarn).
- Setup thêm những extensions thông dụng:
  - ES7+ React/Redux/React-Native snippets
  - Prettier
  - Auto Rename Tag
  - ESLint

## II. Cấu trúc folder dự án

### 1. public

- Thư mục chứa các tập tin tĩnh như hình ảnh, favicon, robots.txt...

### 2. src

- Chứa toàn bộ mã nguồn của ứng dụng ReactJs. Nó chứa các thành phần React, các file CSS, các file test,....
- `main.jsx` là file React chính, để khởi tạo Root, render và xử lý những thuộc tính toàn dự án (global)

### 3. src/components

- Chứa những component có thể tái sử dụng được trong dự án

### 4. src/configs

- Chứa những file config dự án, những file đặc biệt liên quan đến các
  cấu hình của dự án (routes,...)

### 5. src/pages

- Chứa những component đảm nhận vai trò layout của một page
- page/components: component dùng riêng cho page đó

### 6. src/layout

- Chứa các component dạng layout

### 7. src/constant

- Những biến dạng const sử dụng trong toàn hệ thống (PATH, API, message,....)

### 8. src/services

- Các service chứa các lệnh gọi api

### 9. src/assets

- Chứa các file assets chung, chỉ được sử dụng trong src
- Có 1 file style chứa các biến chung, tất cả các component phải import file này

### 10. src/hooks

- Chứa các custom hook

### 11. src/store

- Chứa những Actions, Reducer của Redux

### 12. src/types

- Chứa những file typescript chỉ định, định nghĩa type/model cho các module và dự án

### 13. src/utils

- Chứa những helper functions được tái sử dụng trong dự án

## III. Điều chỉnh file package.json dự án

- Thêm những package cơ bản ban đầu sẽ cần dùng tới trong dự án vào file `src/package.json`.
- Những package như `axios`, `classnames`, `lodash`,...

```jsx
{
    ...
    "dependencies": {
     "@svgr/rollup": "^8.1.0",
    "@types/lodash": "^4.17.6",
        "axios": "^1.6.8",
        "classnames": "^2.5.1",
        "js-cookie": "^3.0.5",
        "lodash": "^4.17.21",
        "query-string": "^9.0.0",
        "react-dom": "^18.2.0",
        "react-hook-form": "^7.51.2"
    },
    "devDependencies": {
        "@types/react-dom": "^18.2.22",
        "@typescript-eslint/eslint-plugin": "^7.2.0",
        "@typescript-eslint/parser": "^7.2.0",
        "@vitejs/plugin-react": "^4.2.1",
        "eslint": "^8.57.0",
        "eslint-plugin-react-hooks": "^4.6.0",
        "eslint-plugin-react-refresh": "^0.4.6",
        "typescript": "^5.2.2",
        "vite": "^5.2.0"
    }
    ...
}
```

<br />
<br />

## Thêm cấu hình điều chỉnh import alias:

- Trong file `tsconfig.json` thêm các cấu hình:

```jsx
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "paths": {
      "@/*": ["src/*"]
    },
    "baseUrl": "."
  },
  "include": ["src"]
}

```

- Trong file `vite.config.ts`:

```jsx
import { defineConfig } from "vite";
import svgr from "@svgr/rollup";
import react from "@vitejs/plugin-react";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react(), svgr()],
  resolve: {
    alias: {
      "@": "/src",
    },
  },
});
```
