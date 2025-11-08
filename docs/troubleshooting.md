# 故障排查指南

## 常见问题

### 1. 无法访问后台

#### 症状
配置后台白名单后无法访问后台管理界面。

#### 原因
当前IP不在白名单中。

#### 解决方案

**方案A：通过数据库清空白名单**
```sql
UPDATE typecho_options 
SET value = REPLACE(value, '"adminWhitelist";s:', '"adminWhitelist";s:0:"') 
WHERE name = 'plugin:BlockIPForTypecho';
```

**方案B：通过SSH/FTP修改配置**
1. 连接服务器
2. 备份数据库
3. 执行上述SQL语句

**方案C：禁用插件**
```sql
UPDATE typecho_options 
SET value = 'a:0:{}' 
WHERE name = 'plugins';
```

**方案D：删除插件文件**
```bash
mv usr/plugins/BlockIPForTypecho usr/plugins/BlockIPForTypecho.bak
```

### 2. 插件启用失败

#### 症状
点击启用按钮后显示错误信息。

#### 可能原因
- PHP版本过低
- 缺少必需扩展
- 数据库权限不足
- 文件权限问题

#### 解决方案

**检查PHP版本**
```bash
php -v
```
要求：PHP 7.4+

**检查PHP扩展**
```bash
php -m | grep -E "pdo|mbstring|json|session"
```

**检查数据库权限**
```sql
SHOW GRANTS FOR CURRENT_USER;
```
需要：CREATE, ALTER, DROP权限

**检查文件权限**
```bash
ls -la usr/plugins/BlockIPForTypecho/
```
建议：755目录，644文件

### 3. 自检报告显示错误

#### 症状
插件启用后显示自检错误或警告。

#### 常见错误

**错误1：数据库表创建失败**
```
原因：数据库权限不足
解决：授予CREATE TABLE权限
```

**错误2：IP地理位置库加载失败**
```
原因：文件缺失或权限问题
解决：检查ip2region目录和文件
```

**错误3：写入权限不足**
```
原因：目录权限问题
解决：chmod 755 usr/plugins/BlockIPForTypecho/
```

### 4. 白名单不生效

#### 症状
配置了白名单但仍然可以访问。

#### 可能原因
- 配置未保存
- IP格式错误
- 缓存问题
- 配置了0.0.0.0

#### 解决方案

**检查配置**
```sql
SELECT value FROM typecho_options 
WHERE name = 'plugin:BlockIPForTypecho';
```

**清除缓存**
```bash
# 清除PHP缓存
service php-fpm restart

# 清除浏览器缓存
Ctrl + Shift + Delete
```

**验证IP格式**
```
正确：192.168.1.100
错误：192.168.1.100.
```

### 5. 黑名单不生效

#### 症状
添加到黑名单的IP仍然可以访问。

#### 可能原因
- IP在白名单中
- IP格式错误
- 工作模式不正确

#### 解决方案

**检查优先级**
```
优先级：白名单 > 黑名单
解决：从白名单中移除该IP
```

**检查工作模式**
```
黑名单模式：黑名单生效
白名单模式：黑名单不生效
智能模式：黑名单生效
```

### 6. 访客日志不记录

#### 症状
启用访客日志但没有记录。

#### 可能原因
- 未启用访客日志
- 访问的是后台
- 被识别为机器人

#### 解决方案

**检查配置**
```
☑ 启用访客日志记录
```

**检查访问路径**
```
前台访问：会记录
后台访问：不记录
```

**检查User-Agent**
```
包含bot关键词：不记录
正常浏览器：会记录
```

### 7. 验证码不显示

#### 症状
登录页面没有显示验证码。

#### 可能原因
- 未启用验证码功能
- GD库未安装
- Session问题

#### 解决方案

**检查配置**
```
☑ 启用登录验证码保护
```

**检查GD库**
```bash
php -m | grep gd
```

**检查Session**
```bash
# 检查session目录权限
ls -la /var/lib/php/sessions/
```

### 8. 性能问题

#### 症状
网站访问速度变慢。

#### 可能原因
- 访客日志过多
- 访问间隔设置过小
- 数据库查询慢

#### 解决方案

**清理日志**
```sql
DELETE FROM typecho_blockip_visitor_log 
WHERE visit_time < DATE_SUB(NOW(), INTERVAL 30 DAY);
```

**优化配置**
```
访问间隔：10秒以上
日志保留天数：30天
□ 禁用访客日志（如不需要）
```

**优化数据库**
```sql
OPTIMIZE TABLE typecho_blockip_visitor_log;
OPTIMIZE TABLE typecho_blockip_access_log;
```

### 9. 地理位置显示错误

#### 症状
IP地理位置显示不准确。

#### 原因
IP地理位置库数据有限。

#### 解决方案
```
这是正常现象，IP地理位置库无法100%准确。
可以更新ip2region库到最新版本。
```

### 10. 误拦截正常访问

#### 症状
正常用户被拦截。

#### 可能原因
- 智能检测过于严格
- 访问频率过快
- 触发敏感词

#### 解决方案

**调整访问间隔**
```
访问间隔：15秒（放宽限制）
```

**添加到白名单**
```
IP白名单：
203.0.113.100       # 正常用户IP
```

**调整敏感词**
```
删除过于宽泛的敏感词
```

## 错误代码

### E001: 数据库连接失败
```
原因：数据库配置错误
解决：检查config.inc.php中的数据库配置
```

### E002: 表创建失败
```
原因：权限不足
解决：授予CREATE TABLE权限
```

### E003: 文件读取失败
```
原因：文件权限或路径问题
解决：检查文件权限和路径
```

### E004: Session启动失败
```
原因：Session配置问题
解决：检查php.ini中的session配置
```

### E005: IP获取失败
```
原因：服务器配置问题
解决：检查$_SERVER变量
```

## 日志分析

### 查看错误日志

**PHP错误日志**
```bash
tail -f /var/log/php-fpm/error.log
```

**服务器错误日志**
```bash
tail -f /var/log/nginx/error.log
tail -f /var/log/apache2/error.log
```

**插件日志**
```bash
grep "BlockIPForTypecho" /var/log/php-fpm/error.log
```

### 常见日志信息

**正常日志**
```
BlockIPForTypecho Debug: 检查IP 203.0.113.100, 模式: smart
```

**拦截日志**
```
BlockIPForTypecho Admin access denied: IP 198.51.100.10 not in whitelist
```

**错误日志**
```
BlockIPForTypecho Error: Database connection failed
```

## 调试技巧

### 启用调试模式
```
调试模式：开启
```

### 查看详细日志
```bash
tail -f /var/log/php-fpm/error.log | grep BlockIPForTypecho
```

### 测试IP匹配
```php
// 在控制台执行
$ip = '192.168.1.100';
$rules = "192.168.1.0/24\n10.0.0.0/8";
$result = IPAccessControl::matchIPRules($ip, $rules, $config);
var_dump($result);
```

### 检查配置
```sql
SELECT * FROM typecho_options 
WHERE name = 'plugin:BlockIPForTypecho';
```

## 性能优化

### 数据库优化
```sql
-- 添加索引
ALTER TABLE typecho_blockip_access_log ADD INDEX idx_ip (ip);
ALTER TABLE typecho_blockip_visitor_log ADD INDEX idx_visit_time (visit_time);

-- 清理旧数据
DELETE FROM typecho_blockip_visitor_log 
WHERE visit_time < DATE_SUB(NOW(), INTERVAL 90 DAY);

-- 优化表
OPTIMIZE TABLE typecho_blockip_access_log;
OPTIMIZE TABLE typecho_blockip_visitor_log;
```

### 配置优化
```
# 减少日志记录
□ 禁用访客日志记录

# 增加访问间隔
访问间隔：15秒

# 减少日志保留
日志保留天数：7天
```

### 服务器优化
```bash
# 增加PHP内存
memory_limit = 256M

# 增加最大执行时间
max_execution_time = 60

# 启用OPcache
opcache.enable=1
```

## 恢复操作

### 恢复默认配置
```sql
UPDATE typecho_options 
SET value = 'a:0:{}' 
WHERE name = 'plugin:BlockIPForTypecho';
```

### 清空所有日志
```sql
TRUNCATE TABLE typecho_blockip_access_log;
TRUNCATE TABLE typecho_blockip_visitor_log;
TRUNCATE TABLE typecho_blockip_auto_blacklist_log;
```

### 重置数据库表
```sql
DROP TABLE IF EXISTS typecho_blockip_access_log;
DROP TABLE IF EXISTS typecho_blockip_visitor_log;
DROP TABLE IF EXISTS typecho_blockip_auto_blacklist_log;
```
然后重新启用插件。

### 完全卸载
1. 禁用插件
2. 删除插件文件
3. 删除数据库表
```sql
DROP TABLE IF EXISTS typecho_blockip_access_log;
DROP TABLE IF EXISTS typecho_blockip_visitor_log;
DROP TABLE IF EXISTS typecho_blockip_auto_blacklist_log;
```

## 获取帮助

### 检查清单
- [ ] 查看错误日志
- [ ] 检查PHP版本和扩展
- [ ] 验证数据库权限
- [ ] 检查文件权限
- [ ] 清除缓存
- [ ] 测试配置

### 提供信息
报告问题时请提供：
- Typecho版本
- PHP版本
- 数据库类型和版本
- 插件版本
- 错误信息
- 相关日志
- 配置信息（隐藏敏感数据）

### 联系支持
- GitHub Issues
- 插件作者邮箱
- Typecho官方论坛

## 预防措施

### 配置前
- 备份数据库
- 记录当前IP
- 准备应急方案
- 在测试环境测试

### 配置时
- 逐步配置
- 及时测试
- 保留备份窗口
- 记录变更

### 配置后
- 验证功能
- 监控日志
- 定期检查
- 更新文档

## 下一步

解决问题后，建议查看：
- [安全最佳实践](./security-best-practices.md)
- [配置指南](./configuration.md)
- [后台白名单使用指南](./admin-whitelist.md)
