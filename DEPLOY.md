# 域名转发系统 - PHP 8.x 部署指南

## 服务器信息

| 项目 | 值 |
|------|-----|
| 服务器IP | 47.119.130.122 |
| SSH端口 | 22 |
| 用户名 | root |
| 密码 | Fj445397@ |
| 网站目录 | /www/wwwroot/url |

## 数据库信息

| 项目 | 值 |
|------|-----|
| 数据库名 | go_139_ink |
| 用户名 | go_139_ink |
| 密码 | k1inSit2drSHEM2p |
| 备份文件 | go_139_ink_2026-02-14_18-14-59_mysql_data_txx3O.sql |

## 部署步骤

### 步骤 1: 上传代码

使用以下任一方法上传代码到服务器：

**方法 A: 使用 WinSCP**
1. 下载安装 WinSCP
2. 连接: `sftp://root@47.119.130.122:22`
3. 密码: `Fj445397@`
4. 将 `D:\ai\url\` 目录下的所有文件上传到 `/www/wwwroot/url/`

**方法 B: 使用 pscp (Windows命令行)**
```cmd
pscp -r -P 22 -pw Fj445397@ -unsafe D:\ai\url\* root@47.119.130.122:/www/wwwroot/url/
```

**方法 C: 使用 scp (Linux/Mac)**
```bash
scp -r -P 22 D:\ai\url\* root@47.119.130.122:/www/wwwroot/url/
```

### 步骤 2: 导入数据库

在服务器上执行:

```bash
# SSH连接到服务器
ssh root@47.119.130.122 -p 22
# 密码: Fj445397@

# 导入数据库
mysql -u go_139_ink -p go_139_ink < go_139_ink_2026-02-14_18-14-59_mysql_data_txx3O.sql

# 或者使用phpMyAdmin导入备份文件
```

### 步骤 3: 设置目录权限

在服务器上执行:

```bash
cd /www/wwwroot/url

# 设置目录权限
chown -R www:www ./

# 确保模板编译目录可写
chmod -R 777 templates_c
chmod -R 777 cache

# 设置基本权限
chmod -R 755 .
```

### 步骤 4: 重启Web服务

```bash
# 重启PHP-FPM (如果使用宝塔面板，可能需要通过面板重启)
systemctl restart php-fpm

# 重启Nginx
systemctl restart nginx

# 或者清理PHP缓存
rm -rf templates_c/*
rm -rf cache/*
```

## 验证部署

1. 访问 `http://47.119.130.122` 或绑定的域名
2. 尝试登录用户中心 (邮箱: `ok@590.net`)
3. 测试域名转发功能

## PHP版本要求

- **最低版本**: PHP 8.0
- **推荐版本**: PHP 8.2 或 8.3

## 包含的现代化改进

1. **PDO预处理语句** - 防止SQL注入
2. **password_hash()** - 现代密码哈希
3. **hash_equals()** - 恒定时间比较(防止时序攻击)
4. **随机数生成** - random_int() 替代 rand()
5. **严格类型** - 使用类型声明

## 目录结构

```
/www/wwwroot/url/
├── admin/              # 后台管理
│   ├── check_login.php
│   ├── forward-list.php
│   ├── user-list.php
│   ├── user-edit.php
│   └── user-delete.php
├── user/               # 用户中心
│   ├── check_login.php
│   ├── login.php
│   ├── forward-add.php
│   ├── forward-edit.php
│   ├── forward-delete.php
│   ├── forward-list.php
│   ├── password-edit.php
│   └── mail-active.php
├── inc/                # 包含文件
│   ├── function.php
│   └── PHPMailer/      # (需要安装PHPMailer v6.x)
├── libs/               # Smarty模板引擎 (需要升级到v4.x)
├── templates/          # 模板文件
├── templates_c/       # 编译缓存
├── cache/             # 缓存目录
├── config.new.php     # 数据库配置
├── config.ser.php     # 系统配置(已存在)
├── main.inc.php       # 主入口
├── jump.php           # 转发核心
├── register.php       # 注册
├── captcha.php        # 验证码
└── index.php          # 首页
```

## 安装第三方库

需要安装 PHPMailer v6.x 和 Smarty v4.x:

```bash
# 在服务器上执行

# 1. 安装PHPMailer
cd /www/wwwroot/url/inc
git clone https://github.com/PHPMailer/PHPMailer.git
# 或下载: https://github.com/PHPMailer/PHPMailer/archive/refs/tags/v6.9.1.zip

# 2. 安装Smarty
cd /www/wwwroot/url/libs
git clone https://github.com/smarty-php/smarty.git
# 或下载: https://github.com/smarty-php/smarty/archive/refs/tags/v4.5.4.zip
```

## 常见问题

### Q: 显示 "Database connection failed"
A: 检查 config.new.php 中的数据库配置是否正确

### Q: 验证码不显示
A: 确保GD库已安装: `extension=gd`

### Q: 邮件发送失败
A: 安装 PHPMailer v6.x 并检查 SMTP 配置

### Q: 模板编译错误
A: 清空 templates_c/ 和 cache/ 目录

## 更新日志

### 2026-02-16: 查询参数转发功能

新增查询参数转发功能，支持将访问URL的查询参数（如 `?ref=chatgpt`）透传到目标地址。

**功能说明：**
- 独立开关，与路径转发分开控制
- 启用后：`a.com/?ref=chatgpt` → `b.com/?ref=chatgpt`
- 与路径转发配合：`a.com/path?ref=x` → `b.com/path?ref=x`

**数据库更新：**
```sql
ALTER TABLE `forward` ADD COLUMN `query_mode` tinyint(1) NOT NULL DEFAULT 0 COMMENT '查询参数转发:0=禁用,1=启用' AFTER `path_mode`;
```
