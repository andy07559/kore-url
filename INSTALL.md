# PHP域名转发系统 - 安装与换域名指南

> **2026-02-14 更新**：全新交互式安装向导，填写数据库和管理员信息即可完成安装，无需手动配置！

## 环境要求

- PHP 8.0+ （原生支持，不再需要兼容模式）
- MySQL 5.7+ / MariaDB 10.2+
- Web服务器 (Nginx/Apache)
- 扩展要求：`pdo`、`pdo_mysql`、`gd`、`session`

---

## 一、新安装（推荐）

### 1.1 上传文件

将所有文件上传到网站根目录，如：`/www/wwwroot/url/`

### 1.2 设置目录权限

```bash
chown -R www:www /www/wwwroot/url
chmod -R 755 /www/wwwroot/url
chmod -R 777 /www/wwwroot/url/templates_c
chmod -R 777 /www/wwwroot/url/cache
```

> www:www 为Web服务器用户，根据系统可能为 `nginx:nginx`、`www-data:www-data` 等

### 1.3 创建数据库（可选）

如果数据库不存在，安装向导会自动创建。也可以提前创建：

```sql
CREATE DATABASE `你的数据库名` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER '你的用户名'@'localhost' IDENTIFIED BY '你的密码';
GRANT ALL PRIVILEGES ON `你的数据库名`.* TO '你的用户名'@'localhost';
FLUSH PRIVILEGES;
```

### 1.4 访问安装页面

浏览器访问：`http://你的域名/install.php`

按照向导填写：
1. **数据库配置** - 主机、端口、库名、用户名、密码
2. **管理员账户** - 邮箱和密码

点击「开始安装」即可完成！

### 1.5 首次配置

安装完成后，登录管理员后台 `http://你的域名/admin/`，进入「网站配置」设置：

| 配置项 | 说明 |
|-------|------|
| `home_domain` | 网站主域名（用于验证域名归属） |
| `register_switch` | 是否开放新用户注册 |
| `audit` | 是否需要审核域名 |
| `smtp` | 邮件服务器配置（可选） |

---

## 二、域名转发模式

系统支持 **7种转发模式**：

| 模式 | 值 | 说明 | 适用场景 |
|------|-----|------|---------|
| **显性转发** | 0 | 301永久重定向，地址栏变成目标网址 | SEO权重传递，适合长期使用 |
| **隐性转发** | 1 | iframe嵌入，地址栏不变 | 隐藏目标网址，但部分网站会阻止嵌入 |
| **隐藏式转发** | 6 | iframe嵌入，标签显示"域名"而非"目标地址" | 更好地隐藏目标网址，提升用户体验 |
| **页面停放** | 2 | 显示自定义停放页面 | 域名未启用时展示 |
| **302跳转** | 3 | 302临时重定向，地址栏变成目标网址 | 临时跳转，适合测试或短期活动 |
| **1转多** | 4 | 依次或轮转到多个目标网址，支持访问次数限制 | A/B测试、流量分发 |
| **短域名** | 5 | 跳转到08.ink短链，支持地理位置转发 | 需要按地区分流的用户 |

### 2.1 隐性转发注意事项

部分网站（如 Netflix、Google、淘宝等）会设置 `X-Frame-Options: SAMEORIGIN` 阻止被 iframe 嵌入，导致显示空白。这是目标网站的安全策略，无法通过本系统解决。

建议对这类网站使用 **显性转发** 或 **302跳转**。

### 2.2 隐藏式转发

隐藏式转发是隐性转进的增强版本，功能与隐性转发完全相同（iframe嵌入，地址栏不变），唯一的区别是：

| 特性 | 隐性转发 | 隐藏式转发 |
|------|---------|-----------|
| 目标地址标签 | 显示"目标地址" | 显示"域名" |
| URL 填写框标签 | "如：http://www.example.com" | "如：https://example.com" |
| 转发效果 | iframe 嵌入目标网页 | iframe 嵌入目标网页 |

**优势**：标签显示"域名"而非"目标地址"，用户填写时更直观，心理上更认同这是自己的域名。

### 2.3 1转多模式配置

选择"1转多"模式后，可配置以下选项：

| 配置项 | 说明 |
|-------|------|
| **转发目标** | 添加多个目标网址，每个可设置访问次数限制（0表示不限） |
| **兜底地址** | 所有目标都用完后跳转的网址 |
| **转发模式** | 依次转发（用完一个到下一个）或 轮转（随机选择可用的） |

**示例配置**：
```
转发目标：
┌────────────────────────────────────────┐
│ [http://example1.com] [访问次数:100] │
│ [http://example2.com] [访问次数:100] │
│ [+ 添加转发目标]                       │
└────────────────────────────────────────┘

兜底地址：[http://fallback.com]
转发模式：[依次转发（用完一个到下一个）v]
```

### 2.3 短域名模式配置

选择"短域名(08.ink)"模式后，系统会调用08.ink API创建短链，实现地理位置转发功能。

| 配置项 | 说明 |
|-------|------|
| **目标地址** | 填写一个默认目标网址（如 https://baidu.com） |
| **短码(1-99)** | 输入1-99的数字，系统会自动在08.ink创建对应短链 |

**工作流程**：
```
用户访问 abc.com
        ↓
302跳转到 https://08.ink/88
        ↓
08.ink 根据用户地理位置转发
```

**在08.ink设置地理位置转发**：
1. 登录 `https://08.ink/user/login`
2. 编辑对应的短链
3. 在「Geo Targeting」中添加不同国家/地区的目标URL

**示例配置**：
```
短码: 88
目标地址: https://baidu.com

08.ink Geo Targeting设置:
├── cn: https://baidu.com      # 中国用户
├── us: https://google.com     # 美国用户
├── jp: https://yahoo.co.jp    # 日本用户
└── (其他): https://baidu.com  # 默认
```

**注意事项**：
- 短码范围：1-99（预留号段）
- 每个短码只能被一个域名使用
- 08.ink API调用失败会导致添加失败

---

## 四、会员系统

系统支持完整的会员功能，包括会员等级、到期时间、域名数量限制，以及密钥开通系统。

### 4.1 会员等级

| 等级 | 名称 | 默认域名限制 | 说明 |
|-----|------|------------|------|
| 0 | 普通用户 | 0 | 注册后默认，无法创建域名 |
| 1 | VIP1 | 5 | 基础会员，可创建5个域名 |
| 2 | VIP2 | 20 | 中级会员，可创建20个域名 |
| 3 | VIP3 | 100 | 高级会员，可创建100个域名 |
| 99 | 永久VIP | 0 | 管理员手动开通，无限制 |

### 4.2 开通方式

**方式一：管理员后台手动开通**

1. 管理员登录后台
2. 进入「用户列表」
3. 点击「编辑」用户
4. 设置会员等级、到期时间、域名限制
5. 点击「保存」

**方式二：密钥开通**

管理员可以批量生成 VIP 密钥，用户输入密钥即可开通：

1. 管理员进入「VIP密钥」管理
2. 点击「生成新密钥」
3. 设置会员等级、时长、域名限制
4. 生成密钥并发放给用户
5. 用户在「用户中心」→「激活VIP」中输入密钥开通

### 4.3 密钥管理

**生成密钥**：

```
会员等级：VIP2 (中级会员)
时  长：30天
域名限制：10个
生成数量：5个

点击「生成密钥」即可生成5个 VIP-XXXXXXXXXXXX 格式的密钥
```

**密钥格式**：
- 格式：`VIP-XXXXXXXXXXXX`（12位随机字符）
- 一次性使用：每个密钥只能被一个用户激活
- 激活后绑定用户，无法转让

### 4.4 用户使用流程

1. **注册账号** → 成为普通用户（无法创建域名）
2. **获取密钥** → 从管理员处获得 VIP 密钥
3. **激活会员** → 用户中心 → 激活VIP → 输入密钥
4. **开始使用** → 获得对应的会员权限

### 4.5 到期处理

- **非永久VIP**：到期后自动降级为普通用户，无法继续创建域名
- **已创建域名**：降级后仍可正常访问和管理（只读）
- **续费**：管理员可以手动延长到期时间

### 4.6 数据库结构

新安装系统会自动创建以下字段和表：

```sql
-- 用户表新增字段
ALTER TABLE `user` ADD COLUMN `vip_level` INT(11) DEFAULT 0 COMMENT '会员等级';
ALTER TABLE `user` ADD COLUMN `vip_expire` INT(11) DEFAULT 0 COMMENT '到期时间戳';
ALTER TABLE `user` ADD COLUMN `domain_limit` INT(11) DEFAULT 0 COMMENT '域名限制';

-- 会员密钥表
CREATE TABLE `vip_key` (
  `id` INT(11) AUTO_INCREMENT,
  `key` VARCHAR(64) UNIQUE COMMENT '密钥',
  `vip_level` INT(11) DEFAULT 1 COMMENT '会员等级',
  `vip_days` INT(11) DEFAULT 0 COMMENT '天数(0=永久)',
  `domain_limit` INT(11) DEFAULT 0 COMMENT '域名限制(0=无限制)',
  `used` TINYINT(1) DEFAULT 0 COMMENT '是否使用',
  `used_by` INT(11) COMMENT '使用者ID',
  `used_time` INT(11) COMMENT '使用时间',
  `create_time` INT(11) COMMENT '创建时间',
  `create_by` INT(11) COMMENT '创建者',
  PRIMARY KEY (`id`)
);
```

---

## 五、HTTPS 转发配置

系统已配置 **通配符 SSL 证书** `*.你的主域名.com`，支持所有子域名自动 HTTPS 访问。

### 5.1 工作原理

```
用户访问 https://子域名.主域名.com
        ↓
Nginx 使用通配符证书解密（无需额外配置）
        ↓
转发到 PHP (jump.php)
        ↓
根据数据库配置跳转到目标网址
```

### 5.2 子域名 HTTPS 支持

| 域名类型 | 是否自动支持 HTTPS | 是否需要数据库记录 |
|---------|------------------|------------------|
| 主域名子域名（如 `*.590.net`） | ✅ 自动支持 | ✅ 需要添加 |
| 外部域名（如 `abc.com`） | ❌ 需单独申请证书 | ✅ 需要添加 |

**示例**：
- `tv.590.net` → ✅ 自动 HTTPS，需添加数据库记录
- `t.590.net` → ✅ 自动 HTTPS，需添加数据库记录
- `abc.com` → ❌ 需单独申请证书，需添加数据库记录

### 5.3 证书管理

#### 自动续期

Let's Encrypt 证书有效期 90 天，系统已配置自动续期：

```bash
# 查看续期定时任务
systemctl list-timers | grep certbot

# 测试自动续期（不实际执行）
certbot renew --dry-run

# 手动强制续期（证书快过期时）
certbot renew
```

#### 续期后重载 Nginx

```bash
nginx -s reload
```

或配置续期后自动重载：

```bash
mkdir -p /etc/letsencrypt/renewal-hooks/post
cat > /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh << 'EOF'
#!/bin/bash
nginx -s reload
EOF
chmod +x /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh
```

### 5.4 外部域名配置 HTTPS

对于非主域名子域名的外部域名，需要单独申请 SSL 证书：

**方式一：使用 Certimate（已部署）**
1. 访问 https://ssl.139.ink
2. 使用管理员账号登录
3. 申请域名证书

**方式二：使用宝塔面板**
1. 进入「网站」→「SSL」→「Let's Encrypt」
2. 选择域名，申请证书

### 5.5 腾讯云 DNS API 配置（如需重新申请通配符证书）

系统使用腾讯云 DNS API 自动验证域名所有权：

```bash
# 配置文件位置
/etc/letsencrypt/tencent-cloud.ini

# 格式
dns_tencentcloud_secret_id = 你的SecretId
dns_tencentcloud_secret_key = 你的SecretKey
dns_tencentcloud_region = ap-shanghai
```

**申请通配符证书命令**：
```bash
certbot certonly \
  --authenticator dns-tencentcloud \
  --dns-tencentcloud-credentials /etc/letsencrypt/tencent-cloud.ini \
  --dns-tencentcloud-propagation-seconds 60 \
  -d 你的主域名.com \
  -d *.你的主域名.com \
  --email admin@你的域名.com \
  --agree-tos \
  --non-interactive
```

---

## 六、换域名

### 6.1 准备工作

确保新域名已解析到服务器IP，且SSL证书已配置（建议）。

### 6.2 通过后台修改配置

换域名最简单的方是：

1. 将网站文件复制到新目录
2. 访问 `http://新域名/install.php`
3. 安装时填写**相同的数据库信息**
4. 安装完成后，登录后台修改「网站配置」中的主域名

### 6.3 Nginx/Apache 配置

#### Nginx 示例（宝塔面板）

```
server {
    listen 80;
    server_name 新域名;
    root /www/wwwroot/url;
    index index.php;

    include enable-php-83.conf;  # PHP版本

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
}
```

#### 旧域名重定向（可选）

```nginx
server {
    listen 80;
    server_name 旧域名;
    return 301 http://新域名$request_uri;
}
```

### 6.4 清理缓存

```bash
rm -rf /www/wwwroot/url/templates_c/*
rm -rf /www/wwwroot/url/cache/*
```

### 6.5 更新域名解析

在域名服务商处，将旧域名的A记录修改为新域名（如果IP不变可跳过）。

### 6.6 验证检查清单

- [ ] 新域名可访问首页
- [ ] 用户可正常注册/登录
- [ ] 转发功能正常工作
- [ ] 管理员后台可访问
- [ ] 邮件发送功能正常（如配置了SMTP）

---

## 七、常见问题

### Q1: 500错误

- 检查 `templates_c` 和 `cache` 目录权限是否为 777
- 检查 PHP 错误日志 `/www/wwwlogs/url.08.ink.error.log`

### Q2: 安装时数据库连接失败

- 确认数据库用户名和密码正确
- 确认用户有创建数据库的权限（或提前手动创建数据库）
- 确认数据库主机地址正确（本地数据库用 `localhost`）

### Q3: 邮件发送失败

- SMTP 配置可在后台「网站配置」中设置
- 注意使用授权码而非登录密码

### Q4: 转发不生效

- 确认域名已正确解析到服务器
- 使用 `ping 域名` 验证 DNS 是否生效
- 检查域名是否在 `home_domain` 配置的域名列表中

### Q5: 从旧版升级后部分功能异常

- 清理缓存：`rm -rf templates_c/* cache/*`
- 如果密码是旧版 MD5，首次登录后系统会自动升级为现代加密

### Q6: 会员系统相关

**Q6.1: 用户显示"您还不是会员"但无法添加域名**

- 默认新注册用户是普通用户（域名限制为0），无法创建域名
- 管理员需要在后台生成 VIP 密钥，用户激活后获得创建域名的权限
- 或管理员手动在用户编辑中开通会员

**Q6.2: 会员到期后会发生什么**

- 到期后自动降级为普通用户
- 已创建的域名仍然可以正常访问（只读）
- 需要续费（管理员延长到期时间）或重新激活密钥才能继续创建域名

**Q6.3: 会员等级说明**

| 等级 | 名称 | 默认域名限制 |
|-----|------|-------------|
| 0 | 普通用户 | 0（不可创建） |
| 1 | VIP1 | 5 |
| 2 | VIP2 | 20 |
| 3 | VIP3 | 100 |
| 99 | 永久VIP | 无限制 |

**Q6.4: 升级会员系统数据库**

如果是从旧版本升级，需要执行数据库升级。访问 `http://你的域名/upgrade.php` 点击"执行数据库升级"按钮即可。

### Q6: 升级到"1转多"模式后报错

如果升级后出现 `Unknown column 'multi_config'` 错误，需要执行数据库迁移：

```sql
ALTER TABLE `forward` ADD COLUMN `multi_config` TEXT DEFAULT NULL AFTER `url`;
```

或在宝塔面板 phpMyAdmin 中手动添加该列。

### Q7: HTTPS 证书相关问题

**Q7.1: 如何让域名支持 HTTPS 转发？**

对于主域名的子域名（如 `*.590.net`），系统已配置通配符证书，自动支持 HTTPS。

对于外部域名（如 `abc.com`），需要单独申请 SSL 证书：

1. 使用 Certimate 申请证书：https://ssl.139.ink
2. 或在宝塔面板「网站」→「SSL」→「Let's Encrypt」申请

**Q7.2: 证书过期怎么办？**

Let's Encrypt 证书有效期 90 天，系统已配置自动续期：

```bash
# 查看续期定时任务
systemctl list-timers | grep certbot

# 手动测试续期
certbot renew --dry-run

# 手动强制续期（如果快过期）
certbot renew
```

**Q7.3: 续期后需要重启 Nginx**

```bash
nginx -s reload
```

或配置续期后自动重载：

```bash
cat > /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh << 'EOF'
#!/bin/bash
nginx -s reload
EOF
chmod +x /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh
```

---

## 八、从旧版升级

### 8.1 升级到 PHP 8.x

本系统已原生支持 **PHP 8.0+**，升级步骤：

1. 备份数据库和网站文件
2. 将新版本文件上传覆盖
3. 访问 `http://你的域名/install.php` 重新安装（使用相同数据库）
4. 清理缓存：`rm -rf templates_c/* cache/*`
5. 测试所有功能

> 注意：旧版 `config.new.php` 已不再需要，配置信息全部存储在 `config.ser.php`

---

## 九、目录结构

```
/www/wwwroot/url/
├── admin/              # 管理员后台
│   ├── index.php      # 管理后台首页
│   ├── user-list.php  # 用户列表
│   ├── user-edit.php  # 用户编辑
│   ├── forward-list.php
│   └── config-ser.php # 网站配置
├── user/              # 用户中心
│   ├── index.php
│   ├── forward-list.php
│   ├── forward-add.php
│   ├── forward-edit.php
│   ├── password-edit.php
│   └── login.php
├── templates/         # Smarty模板
│   ├── admin_*.tpl
│   ├── user_*.tpl
│   └── header.tpl / footer.tpl
├── libs/              # 第三方库
│   ├── Smarty/        # Smarty 4.x
│   └── PHPMailer/     # PHPMailer 6.x
├── inc/               # 核心函数
│   ├── function.php
│   └── ...
├── config.new.php     # 数据库连接（安装时配置）
├── config.ser.php     # 系统配置（序列化）
├── main.inc.php       # 主配置
├── jump.php           # 转发核心逻辑
├── install.php        # 安装向导（Web）
├── install_cli.php    # 安装脚本（CLI）
└── security.php       # 安全检查
```

---

## 七、备份与恢复

### 备份

```bash
# 备份数据库
mysqldump -u 用户名 -p 数据库名 > backup_$(date +%Y%m%d).sql

# 备份文件
zip -r url_backup_$(date +%Y%m%d).zip /www/wwwroot/url
```

### 恢复

```bash
# 恢复数据库
mysql -u 用户名 -p 数据库名 < backup_xxx.sql

# 恢复文件（覆盖后需清理缓存）
unzip -o url_backup_xxx.zip -d /
rm -rf /www/wwwroot/url/templates_c/*
```
