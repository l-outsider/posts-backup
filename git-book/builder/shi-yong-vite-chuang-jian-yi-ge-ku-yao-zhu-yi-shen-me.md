# 使用 Vite 创建一个库要注意什么

记录时间 2024 年 10 月 22 日

参考文章 [<mark style="color:orange;">https://dev.to/receter/how-to-create-a-react-component-library-using-vites-library-mode-4lma</mark>](https://dev.to/receter/how-to-create-a-react-component-library-using-vites-library-mode-4lma)

## 第一步 创建一个项目

{% code fullWidth="false" %}
```sh
pnpm create vite my-vue-app --template react-ts
```
{% endcode %}

## 第二步 配置 package.json

要注意的是在现代的配置文件中可以提供 `main` \ `module` \ `export` 字段。

```json
{
  "name": "my-lib",
  "type": "module",
  "files": ["dist"],
  "main": "./dist/my-lib.umd.cjs",
  "module": "./dist/my-lib.js",
  "exports": {
    ".": {
      "import": "./dist/my-lib.js",
      "require": "./dist/my-lib.umd.cjs"
    }
  }
}
```

如果是多入口

```json
"exports": {
    ".": {
      "import": "./dist/my-lib.js",
      "require": "./dist/my-lib.cjs"
    },
    "./secondary": {
      "import": "./dist/secondary.js",
      "require": "./dist/secondary.cjs"
    }
  }
```

<details>

<summary>Example</summary>

```json
{
  "name": "@scope/base-component-react",
  "private": false,
  "version": "0.0.1-beta.1",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint .",
    "preview": "vite preview"
  },
  "dependencies": {
    "i18next": "^23.15.1",
    "i18next-browser-languagedetector": "^8.0.0",
    "react": "^18.3.1",
    "react-dom": "^18.3.1"
  },
  "devDependencies": {
    "@tencentcloud/eslint-config-hybrid": "workspace: *",
    "@types/react": "^18.3.10",
    "@types/react-dom": "^18.3.0",
    "@vitejs/plugin-react": "^4.3.2",
    "eslint": "^8.57.1",
    "globals": "^15.9.0",
    "sass": "^1.80.3",
    "typescript": "^5.5.3",
    "vite": "^5.4.8",
    "vite-plugin-dts": "^4.2.2"
  }
}
```

</details>

