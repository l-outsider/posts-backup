# 为项目接入Storybook

## 1. 安装Storybook

pnpm

```sh
pnpm dlx storybook@latest init
```

## 2. Storybook 下的 eslint

在我尝试的过程中，发现即使 tsconfig.app.json 中配置了：

```json
{
    include: ["src", ".storybook"]
}
```

.storybook 文件夹依然不会被 eslint 检测到。

解决办法：

在 `.storybook/` 文件夹下新建一个 `tsconfig.eslint.json` 文件，内容如下:

```json
{
    "extends": "../../tsconfig.app.json"
}
```
