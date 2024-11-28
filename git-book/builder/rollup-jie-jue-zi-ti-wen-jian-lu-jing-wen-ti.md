---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# rollup解决字体文件路径问题

在 react 使用字体文件时，src 中的源码结构是这样的：

> 使用 gpt 生成：我记得有个字符就是横线+竖线，换成这个字符

{% code fullWidth="false" %}
```
├── src
│  ├── assets
│  │  └── fonts
│  │      └── iconfont.woff
│  └── styles
│     └── fonts
│         └── iconfont.scss
```
{% endcode %}

其中 iconfont.scss 的内容是：

{% code title="iconfont.scss" %}
```scss
@font-face {
  font-family: iconfont;
  src:
    // 此处的路径是对的
    url("../../assets/fonts/iconfont.woff2?t=1722856322648") format("woff2"),
    url("../../assets/fonts/iconfont.woff?t=1722856322648") format("woff"),
    url("../../assets/fonts/iconfont.ttf?t=1722856322648") format("truetype");
}
```
{% endcode %}

此时的路径是没问题的，但是在构建打包之后，发现 dist 目录中没有字体文件。

## 第一步 使用 rollup-plugin-copy 拷贝字体文件 <a href="#gqbol" id="gqbol"></a>

省略了部分内容之后，核心如下所示，使用 copy 插件，将 `src/assets/fonts/*` 下的内容拷贝到 `dist/assets/fonts` 内部。

```javascript
import copy from 'rollup-plugin-copy';

export default [
  {
    ...getBaseConfig(),
    plugins: [
      postcss({
        extract: true,
        minimize: true,
        plugins: [
          autoprefixer(),
        ],
      }),
      // copy font files and ensure paths are correct in npm publish
      copy({
        targets: [{
          src: [
            'src/assets/fonts/iconfont.ttf',
            'src/assets/fonts/iconfont.woff',
            'src/assets/fonts/iconfont.woff2',
          ],
          dest: ['dist/assets/fonts'],
        }],
      }),
    ],
  },
];
```

## 第二步 使用 postcss-url 实现 css 资源路径的替换 <a href="#pzdrs" id="pzdrs"></a>

然而还是会报错，因为构建后的内容结构如下所示

```
├── dist/
│  ├── assets/
│  │  └── fonts/
│  │     └── iconfont.woff
│  └── index.css
```

index.css 中是压缩后的 css 文件，其中关于字体的路径依然是 `"../../assets/fonts/iconfont.woff2?t=1722856322648"`，这就不对了。

因此使用 postcss 的插件`postcss-url`实现字体资源路径的替换（注意这不是 rollup 的插件）。

它的原理是分析 css 中的 `url()`部分，可以拿到资源路径，然后提供一个 callback 函数支持修改该路径。

> [https://github.com/postcss/postcss-url/tree/master?tab=readme-ov-file#url-function](https://github.com/postcss/postcss-url/tree/master?tab=readme-ov-file#url-function)

看一下下面的代码，首先判断 pathname 是不是字体路径，如果不是返回原始 url。

如果是的话，替换为 dist 目录结构中的资源路径。

```javascript
import copy from 'rollup-plugin-copy';
import postcssUrl from 'postcss-url';

export default [
  {
    ...getBaseConfig(),
    plugins: [
      postcss({
        extract: true,
        minimize: true,
        plugins: [
          autoprefixer(),
          postcssUrl({
            url: (pathInfo) => {
              if (!/\.(woff|woff2|ttf)$/.test(pathInfo.pathname)) {
                return pathInfo.url;
              }
              // ensure paths are correct in npm publish
              return `./assets/fonts/${pathInfo.url.split('/').pop()}`;
            },
          }),
        ],
      }),
      // copy font files and ensure paths are correct in npm publish
      copy({
        targets: [{
          src: [
            'src/assets/fonts/iconfont.ttf',
            'src/assets/fonts/iconfont.woff',
            'src/assets/fonts/iconfont.woff2',
          ],
          dest: ['dist/assets/fonts'],
        }],
      }),
    ],
  },
];
```

大功告成！
