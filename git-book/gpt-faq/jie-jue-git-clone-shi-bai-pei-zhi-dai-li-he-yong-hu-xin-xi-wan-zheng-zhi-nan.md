# 解决 Git Clone 失败：配置代理和用户信息完整指南

## 问题描述

在使用 Git 进行项目克隆时，经常会遇到连接超时或者无法访问的情况，特别是在使用 Clash 作为代理工具时。本文将详细介绍如何正确配置 Git，包括基础用户信息设置和代理配置。

## 解决方案

### 1. Git 基础配置

在使用 Git 之前，首先需要配置用户信息。这些信息将用于标识你的提交记录。

#### 1.1 查看当前配置

```bash
# 查看所有配置
git config --list

# 查看用户名和邮箱
git config --global --get user.name
git config --global --get user.email
```

#### 1.2 设置用户信息

```bash
# 设置用户名
git config --global user.name "你的用户名"

# 设置邮箱
git config --global user.email "你的邮箱@example.com"
```

#### 1.3 如何获取正确的用户信息

如果你使用 GitHub：

1. 登录 GitHub 账户
2. 点击右上角头像
3. 进入 Settings（设置）
4. 在左侧菜单中：
   * Public profile 中可查看用户名
   * Emails 选项中可查看邮箱

### 2. 配置代理

如果你使用 Clash 作为代理工具，需要正确配置 Git 的代理设置。

#### 2.1 HTTP/HTTPS 代理设置

```bash
# 设置 HTTP 代理
git config --global http.proxy http://127.0.0.1:7890

# 设置 HTTPS 代理
git config --global https.proxy http://127.0.0.1:7890
```

```bash
git config --global http.https://github.com.proxy http://127.0.0.1:7890
```

#### 2.2 使用 SOCKS5 代理（备选方案）

```bash
git config --global http.proxy socks5://127.0.0.1:7890
git config --global https.proxy socks5://127.0.0.1:7890
```

#### 2.3 验证代理配置

```bash
git config --global --get http.proxy
git config --global --get https.proxy
```

#### 2.4 取消代理设置

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## 故障排除

如果配置完成后仍然遇到问题，可以检查以下几点：

1. **确认 Clash 状态**
   * 确保 Clash 正在运行
   * 检查代理模式（Global 或 Rule）
   * 验证代理端口（默认 7890）
2. **检查网络连接**
   * 确认防火墙设置
   * 测试基本网络连接
3.  **使用详细模式克隆**

    ```bash
    git clone -v <repository-url>
    ```

    这将显示更多的错误信息，有助于诊断问题。

## 总结

正确配置 Git 需要注意两个主要方面：基础用户信息和网络代理设置。确保这两部分都正确配置，可以解决大多数 Git 克隆失败的问题。如果仍然遇到问题，建议查看详细的错误信息，这样可以更准确地定位和解决问题。

## 参考资料

* [Git 官方文档](https://git-scm.com/doc)
* [GitHub 帮助文档](https://docs.github.com)
