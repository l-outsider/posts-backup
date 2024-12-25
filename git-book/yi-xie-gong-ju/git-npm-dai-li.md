# git | npm 代理

明明挂了代理还是 clone 不下来？

```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

git config --global --unset http.proxy
git config --global --unset https.proxy
```

明明挂了代理还是 npm install 不下来？

```bash
# 设置代理
npm config set proxy http://127.0.0.1:7890
npm config set https-proxy http://127.0.0.1:7890

# 如果不需要代理，可以删除代理设置
npm config delete proxy
npm config delete https-proxy
```
