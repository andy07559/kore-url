# 域名转发系统 - 安装教程

## 目录

1. [系统要求](#系统要求)
2. [文件上传](#文件上传)
3. [权限设置](#权限设置)
4. [运行安装程序](#运行安装程序)
5. [安装后配置](#安装后配置)
6. [Nginx配置](#nginx配置)
7. [Apache配置](#apache配置)
8. [SSL证书配置](#ssl证书配置)
9. [故障排查](#故障排查)
10. [安全建议](#安全建议)

---

## 系统要求

### 基础环境

| 组件 | 最低要求 | 推荐配置 |
|------|---------|---------|
| PHP | 8.0+ | 8.1+ |
| MySQL | 5.7+ | 8.0+ |
| Web服务器 | Nginx 1.18+ / Apache 2.4+ | 最新稳定版 |

### PHP扩展要求

| 扩展名 | 状态 | 说明 |
|-------|------|------|
| pdo | 必需 | 数据库连接 |
| pdo_mysql | 必需 | MySQL驱动 |
| session | 必需 | 会话管理 |
| json | 必需 | 数据处理 |
| gd | 推荐 | 验证码生成 |
| mbstring | 推荐 | 多字节字符串处理 |

### 服务器配置建议

- PHP内存限制: 128M+
- 最大执行时间: 30s+
- 文件上传大小: 根据需求调整
- 开启短标签支持: `short_open_tag = On`

---

## 文件上传

### 方式一：FTP/SFTP上传

1. 将项目文件上传到服务器目录，例如：
   ```
   /www/wwwroot/url590/
   ```

2. 确保目录结构完整：
   ```
   url590/
   ├── admin/          # 管理后台
   ├── user/           # 用户中心
   ├── inc/            # 核心函数
   ├── libs/           # Smarty模板引擎
   ├── templates/      # 模板文件
   ├── templates_c/    # 模板编译目录
   ├── cache/          # 缓存目录
   ├── index.php       # 入口文件
   ├── install.php     # 安装程序
   └── ...
   ```

### 方式二：Git部署

```bash
cd /www/wwwroot/
git clone <repository-url> url590
cd url590
```

### 方式三：压缩包上传

1. 本地打包为 zip 文件
2. 上传到服务器
3. 解压缩：
   ```bash
   unzip url590.zip -d /www/wwwroot/
   ```

---

## 权限设置

### Linux权限设置命令

```bash
# 进入项目目录
cd /www/wwwroot/url590

# 设置所有者（根据实际Web用户调整）
chown -R www:www /www/wwwroot/url590

# 设置基本权限
chmod -R 755 /www/wwwroot/url590

# 设置可写目录权限
chmod -R 777 /www/wwwroot/url590/templates_c
chmod -R 777 /www/wwwroot/url590/cache

# 验证权限
ls -la
```

### Windows服务器权限

1. 右键文件夹 → 属性 → 安全
2. 添加 `IIS_IUSRS` 或 `IUSR` 用户
3. 授予以下权限：
   - 读取和执行
   - 列出文件夹内容
   - 读取
4. 对以下目录额外授予写入权限：
   - `templates_c/`
   - `cache/`

### 验证权限

安装程序会自动检测以下目录的写入权限：

- 主目录（写入 config.new.php, config.ser.php）
- templates_c/（模板编译缓存）
- cache/（系统缓存）

---

## 运行安装程序

### 第一步：访问安装页面

在浏览器中访问：
```
http://your-domain.com/install.php
```

### 第二步：环境检测

安装程序会自动检测以下项目：

| 检测项 | 说明 |
|-------|------|
| PHP版本 | 必须是 8.0+ |
| PDO扩展 | 数据库连接必需 |
| GD扩展 | 验证码功能需要 |
| Session扩展 | 会话管理必需 |
| 目录权限 | 配置和缓存目录必须可写 |

如果某项检测失败，请先解决环境问题再继续安装。

### 第三步：填写配置信息

#### 1. 数据库配置

| 字段 | 说明 | 示例 |
|------|------|------|
| 数据库主机 | MySQL服务器地址 | 127.0.0.1 或 localhost |
| 数据库端口 | MySQL端口 | 3306 |
| 数据库名称 | 数据库名（会自动创建） | url_forwarding |
| 数据库用户名 | MySQL用户名 | root |
| 数据库密码 | MySQL密码 | ******** |

#### 2. 系统配置

| 字段 | 说明 | 示例 |
|------|------|------|
| 主域名 | 用于访问后台的域名 | url.example.com |

**注意**：主域名用于访问系统管理界面，其他绑定到该服务器的域名将自动进行转发处理。

#### 3. 管理员账户

| 字段 | 说明 | 要求 |
|------|------|------|
| 管理员邮箱 | 登录用户名 | 有效邮箱格式 |
| 管理员密码 | 登录密码 | 至少6个字符 |

### 第四步：完成安装

点击"开始安装"后，系统会自动：

1. ✅ 创建数据库（如果不存在）
2. ✅ 创建数据表（user, forward, vip_key）
3. ✅ 创建管理员账户
4. ✅ 生成配置文件（config.new.php, config.ser.php）
5. ✅ 创建安装锁定文件（install.lock）

### 安装成功提示

安装成功后会显示：

- 管理员邮箱
- 管理员密码
- 后续操作建议

**请务必保存好管理员账户信息！**

---

## 安装后配置

### 1. 安全操作

```bash
# 删除或重命名安装程序（强烈建议）
rm install.php
# 或
mv install.php install.php.bak

# 设置配置文件只读
chmod 444 config.new.php config.ser.php
```

### 2. 检查域名绑定

确保在服务器上正确配置域名绑定：

- 主域名：指向系统目录
- 转发域名：泛解析或指向同一目录

### 3. 访问管理后台

```
URL: http://your-domain.com/admin/
用户名：安装时设置的邮箱
密码：安装时设置的密码
```

### 4. 基础系统配置

登录后台后，建议进行以下配置：

| 配置项 | 建议值 | 说明 |
|-------|-------|------|
| 注册开关 | 根据需求 | 是否开放用户注册 |
| 域名审核 | 开启 | 新添加的域名需要审核 |
| 腾讯URL保护 | 开启 | 防止域名被拦截 |

### 5. 邮件配置（可选）

如需邮件通知功能，配置SMTP：

| 配置项 | 说明 |
|-------|------|
| SMTP类型 | SSL/TLS |
| SMTP服务器 | 如：smtp.qq.com |
| SMTP端口 | 465 (SSL) 或 587 (TLS) |
| 发件邮箱 | 系统发件邮箱 |
| 发件人名称 | 如：智核分流系统 |

---

## Nginx配置

### 基础配置

```nginx
server {
    listen 80;
    server_name url.example.com;
    root /www/wwwroot/url590;
    index index.php index.html;

    # PHP处理
    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;  # 或 unix:/tmp/php-cgi.sock
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }

    # 禁止访问敏感文件
    location ~ /\.(?:htaccess|htpasswd|ini|log|sh|inc)$ {
        deny all;
    }
}
```

### 泛域名配置（推荐）

```nginx
server {
    listen 80;
    server_name url.example.com *.example.com;
    root /www/wwwroot/url590;
    index index.php index.html;

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # 泛域名支持
    if ($host ~* ^(.*)\.example\.com$) {
        set $subdomain $1;
    }
}
```

### HTTPS配置（推荐）

```nginx
server {
    listen 443 ssl http2;
    server_name url.example.com *.example.com;

    root /www/wwwroot/url590;
    index index.php;

    # SSL证书配置
    ssl_certificate /path/to/your/fullchain.pem;
    ssl_certificate_key /path/to/your/privkey.pem;

    # SSL优化配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # HTTP自动跳转HTTPS
    if ($scheme = http) {
        return 301 https://$server_name$request_uri;
    }
}

# HTTP重定向
server {
    listen 80;
    server_name url.example.com *.example.com;
    return 301 https://$server_name$request_uri;
}
```

### 宝塔面板配置

如果使用宝塔面板：

1. 创建网站
2. 设置网站目录为 `/www/wwwroot/url590`
3. 设置运行目录为 `/`
4. PHP版本选择 8.0+
5. 伪静态选择默认即可
6. 开启SSL（推荐）

---

## Apache配置

### .htaccess配置

在项目根目录创建 `.htaccess` 文件：

```apache
<IfModule mod_rewrite.c>
    RewriteEngine On

    # 禁止访问敏感文件
    <FilesMatch "(^\.htaccess|^\.htpasswd|^\.ini|\.log|\.sh|\.inc)$">
        Order allow,deny
        Deny from all
    </FilesMatch>

    # 禁止访问备份文件
    <FilesMatch "\.(bak|backup|old|sql|zip|tar|gz)$">
        Order allow,deny
        Deny from all
    </FilesMatch>
</IfModule>

# PHP设置（如果允许）
<IfModule mod_php8.c>
    php_value short_open_tag 1
    php_value display_errors Off
    php_value log_errors On
</IfModule>
```

### 虚拟主机配置

```apache
<VirtualHost *:80>
    ServerName url.example.com
    ServerAlias *.example.com
    DocumentRoot "/www/wwwroot/url590"

    <Directory "/www/wwwroot/url590">
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog "logs/url-error.log"
    CustomLog "logs/url-access.log" combined
</VirtualHost>
```

---

## SSL证书配置

### Let's Encrypt免费证书

#### 使用Certbot

```bash
# 安装Certbot
yum install certbot python3-certbot-nginx  # CentOS/RHEL
# 或
apt install certbot python3-certbot-nginx  # Debian/Ubuntu

# 获取证书（泛域名需要DNS验证）
certbot --nginx -d url.example.com -d *.example.com

# 自动续期
certbot renew --dry-run
```

#### 使用宝塔面板

1. 网站设置 → SSL
2. 选择 Let's Encrypt
3. 填写邮箱
4. 点击申请
5. 开启自动续签

### 其他SSL获取方式

| 来源 | 说明 | 推荐场景 |
|------|------|---------|
| Let's Encrypt | 免费，90天有效期 | 个人项目、测试 |
| 阿里云SSL | 付费，1年有效期 | 商业项目 |
| 腾讯云SSL | 付费，1年有效期 | 商业项目 |
| Cloudflare | 免费，灵活 | CDN场景 |

---

## 故障排查

### 1. 安装页面无法访问

**症状**：访问 install.php 显示404或500错误

**排查步骤**：

```bash
# 检查文件是否存在
ls -la /www/wwwroot/url590/install.php

# 检查权限
stat /www/wwwroot/url590/install.php

# 查看错误日志
tail -f /var/log/nginx/error.log
# 或
tail -f /var/log/httpd/error_log
```

**解决方案**：
- 确认文件已上传
- 检查文件权限（644）
- 检查目录权限（755）
- 检查PHP版本

### 2. 数据库连接失败

**症状**：安装时提示"数据库连接失败"

**排查步骤**：

```bash
# 测试MySQL连接
mysql -h 127.0.0.1 -P 3306 -u root -p

# 检查MySQL服务状态
systemctl status mysql
# 或
systemctl status mariadb
```

**解决方案**：
- 确认MySQL服务运行正常
- 检查用户名和密码
- 确认数据库端口正确
- 检查防火墙设置

### 3. 权限写入失败

**症状**：安装时提示"无法写入配置文件"

**解决方案**：

```bash
# 检查目录所有者
ls -la /www/wwwroot/url590/

# 修改所有者
chown -R www:www /www/wwwroot/url590/

# 设置权限
chmod 755 /www/wwwroot/url590/
chmod 777 /www/wwwroot/url590/templates_c/
chmod 777 /www/wwwroot/url590/cache/
```

### 4. 域名转发不生效

**症状**：访问转发域名显示404或不转发

**排查步骤**：

1. 检查域名DNS解析
2. 检查服务器域名绑定
3. 检查数据库中是否有该域名的记录
4. 检查域名状态是否已审核通过
5. 查看jump.php执行日志

**解决方案**：

```bash
# 检查域名解析
nslookup your-domain.com
ping your-domain.com

# 检查Nginx配置
nginx -t
systemctl reload nginx

# 检查数据库记录
mysql -u root -p
USE your_database;
SELECT * FROM forward WHERE domain = 'your-domain.com';
```

### 5. 验证码不显示

**症状**：注册或登录时验证码图片不显示

**解决方案**：

```bash
# 检查GD扩展
php -m | grep gd

# 安装GD扩展
yum install php-gd  # CentOS/RHEL
apt install php-gd   # Debian/Ubuntu

# 重启PHP服务
systemctl restart php-fpm
```

### 6. 后台无法登录

**症状**：输入正确的邮箱和密码后无法登录

**解决方案**：

1. 检查Cookie是否启用
2. 检查浏览器是否支持Session
3. 清除浏览器缓存和Cookie
4. 检查templates_c目录权限
5. 重新生成管理员账户

```sql
-- 重置管理员密码（需要在数据库中执行）
UPDATE user SET pass = '$2y$10$新的hash值' WHERE id = 1;
```

---

## 安全建议

### 1. 安装后安全操作

```bash
# 删除安装程序
rm install.php

# 删除安装SQL文件
rm -rf install/

# 设置配置文件只读
chmod 444 config.new.php config.ser.php
```

### 2. 禁用目录浏览

在Nginx配置中添加：

```nginx
location / {
    autoindex off;
}
```

在Apache配置中添加：

```apache
<Directory "/www/wwwroot/url590">
    Options -Indexes
</Directory>
```

### 3. 限制敏感文件访问

```nginx
# 禁止访问配置文件
location ~ /\.(?!well-known).* {
    deny all;
}

# 禁止访问备份文件
location ~ \.(bak|backup|old|sql|zip|tar|gz)$ {
    deny all;
}
```

### 4. 定期备份

```bash
# 数据库备份
mysqldump -u root -p your_database > backup_$(date +%Y%m%d).sql

# 文件备份
tar -czf backup_$(date +%Y%m%d).tar.gz /www/wwwroot/url590/

# 自动备份脚本
cat > /root/backup_url.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/root/backups"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR
mysqldump -u root -p你的密码 your_database > $BACKUP_DIR/db_$DATE.sql
tar -czf $BACKUP_DIR/files_$DATE.tar.gz /www/wwwroot/url590/
find $BACKUP_DIR -mtime +7 -delete
EOF

chmod +x /root/backup_url.sh

# 添加到定时任务
crontab -e
# 每天凌晨2点备份
0 2 * * * /root/backup_url.sh
```

### 5. 监控和日志

```bash
# 设置日志轮转
cat > /etc/logrotate.d/url590 << 'EOF'
/www/wwwroot/url590/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
}
EOF
```

### 6. 防火墙配置

```bash
# 只开放必要端口
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-port=3306/tcp  # 如果需要远程数据库
firewall-cmd --reload
```

---

## 技术支持

如有问题，请提供以下信息：

1. PHP版本：`php -v`
2. MySQL版本：`mysql -V`
3. 错误日志内容
4. 具体操作步骤

---

**最后更新**: 2024年
**系统版本**: v2.01
