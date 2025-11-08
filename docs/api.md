# API 文档

## 核心类

### AllowAdminIP

后台访问IP白名单管理类。

#### checkAdminAccess()

检查后台访问权限。

**签名：**
```php
public static function checkAdminAccess(string $clientIP, object $config): void
```

**参数：**
- `$clientIP` (string) - 客户端IP地址
- `$config` (object) - 插件配置对象

**返回值：**
- void - 如果IP不在白名单中，将执行重定向并终止脚本

**示例：**
```php
$clientIP = IPAccessControl::getRealClientIP($request);
$config = Widget::widget('Widget_Options')->plugin('BlockIPForTypecho');
AllowAdminIP::checkAdminAccess($clientIP, $config);
```

### IPAccessControl

IP访问控制类。

#### getRealClientIP()

获取真实客户端IP地址。

**签名：**
```php
public static function getRealClientIP(Request $request): string
```

**参数：**
- `$request` (Request) - Typecho请求对象

**返回值：**
- string - 客户端IP地址

**示例：**
```php
$request = new Request();
$clientIP = IPAccessControl::getRealClientIP($request);
```

#### matchIPRules()

匹配IP规则。

**签名：**
```php
public static function matchIPRules(string $ip, string $rules, object $config): bool
```

**参数：**
- `$ip` (string) - 要匹配的IP地址
- `$rules` (string) - IP规则字符串（多行）
- `$config` (object) - 插件配置对象

**返回值：**
- bool - 是否匹配成功

**示例：**
```php
$ip = '192.168.1.100';
$rules = "192.168.1.0/24\n10.0.0.0/8";
$matched = IPAccessControl::matchIPRules($ip, $rules, $config);
```

#### isBlacklisted()

检查IP是否在黑名单中。

**签名：**
```php
public static function isBlacklisted(string $ip, object $config): bool
```

**参数：**
- `$ip` (string) - IP地址
- `$config` (object) - 插件配置对象

**返回值：**
- bool - 是否在黑名单中

#### isWhitelisted()

检查IP是否在白名单中。

**签名：**
```php
public static function isWhitelisted(string $ip, object $config): bool
```

**参数：**
- `$ip` (string) - IP地址
- `$config` (object) - 插件配置对象

**返回值：**
- bool - 是否在白名单中

#### addToBlacklist()

添加IP到黑名单。

**签名：**
```php
public static function addToBlacklist(string $ip, object $config, string $reason): void
```

**参数：**
- `$ip` (string) - IP地址
- `$config` (object) - 插件配置对象
- `$reason` (string) - 拉黑原因

**返回值：**
- void

### SecurityHelper

安全辅助类。

#### isAdminArea()

判断URL是否为后台区域。

**签名：**
```php
public static function isAdminArea(string $url): bool
```

**参数：**
- `$url` (string) - URL地址

**返回值：**
- bool - 是否为后台区域

**示例：**
```php
$url = '/admin/index.php';
$isAdmin = SecurityHelper::isAdminArea($url);
```

### SecurityDetector

安全检测类。

#### detectSQLInjection()

检测SQL注入攻击。

**签名：**
```php
public static function detectSQLInjection(): bool
```

**返回值：**
- bool - 是否检测到SQL注入

#### detectXSS()

检测XSS攻击。

**签名：**
```php
public static function detectXSS(): bool
```

**返回值：**
- bool - 是否检测到XSS攻击

#### detectCSRF()

检测CSRF攻击。

**签名：**
```php
public static function detectCSRF(): bool
```

**返回值：**
- bool - 是否检测到CSRF攻击

### BlockHandler

拦截处理类。

#### blockAccess()

拦截访问并显示403页面。

**签名：**
```php
public static function blockAccess(string $ip, string $reason, object $config): void
```

**参数：**
- `$ip` (string) - 被拦截的IP地址
- `$reason` (string) - 拦截原因
- `$config` (object) - 插件配置对象

**返回值：**
- void - 此方法不会返回，将终止脚本执行

### Logger

日志记录类。

#### logBlockedAccess()

记录拦截日志。

**签名：**
```php
public static function logBlockedAccess(string $ip, string $reason): void
```

**参数：**
- `$ip` (string) - 被拦截的IP地址
- `$reason` (string) - 拦截原因

**返回值：**
- void

#### logAutoBlacklist()

记录自动拉黑日志。

**签名：**
```php
public static function logAutoBlacklist(string $ip, string $reason): void
```

**参数：**
- `$ip` (string) - 被拉黑的IP地址
- `$reason` (string) - 拉黑原因

**返回值：**
- void

#### recordLastAccess()

记录最后访问时间。

**签名：**
```php
public static function recordLastAccess(string $ip, string $url): void
```

**参数：**
- `$ip` (string) - IP地址
- `$url` (string) - 访问的URL

**返回值：**
- void

### GeoLocation

地理位置查询类。

#### getLocation()

获取IP的地理位置信息。

**签名：**
```php
public static function getLocation(string $ip): array
```

**参数：**
- `$ip` (string) - IP地址

**返回值：**
- array - 地理位置信息数组

**示例：**
```php
$location = GeoLocation::getLocation('203.0.113.100');
// 返回: ['country' => '中国', 'province' => '广东省', 'city' => '深圳市']
```

#### isBlockedRegion()

检查IP是否在被禁地区。

**签名：**
```php
public static function isBlockedRegion(string $ip, object $config): bool
```

**参数：**
- `$ip` (string) - IP地址
- `$config` (object) - 插件配置对象

**返回值：**
- bool - 是否在被禁地区

### SmartDetector

智能检测类。

#### isSmartBlocked()

智能检测是否应该拦截。

**签名：**
```php
public static function isSmartBlocked(string $ip, object $config): array
```

**参数：**
- `$ip` (string) - IP地址
- `$config` (object) - 插件配置对象

**返回值：**
- array - 拦截原因数组，空数组表示不拦截

### VisitorStats

访客统计类。

#### logVisitorAccess()

记录访客访问日志。

**签名：**
```php
public static function logVisitorAccess(string $ip, string $url): void
```

**参数：**
- `$ip` (string) - 访客IP地址
- `$url` (string) - 访问的URL

**返回值：**
- void

#### isBot()

判断是否为机器人。

**签名：**
```php
public static function isBot(string $ip, string $userAgent): bool
```

**参数：**
- `$ip` (string) - IP地址
- `$userAgent` (string) - User-Agent字符串

**返回值：**
- bool - 是否为机器人

## 钩子

### admin/common.php - begin

后台页面开始加载时触发。

**注册：**
```php
TypechoPlugin::factory('admin/common.php')->begin = [__CLASS__, 'checkIPAccess'];
```

**用途：**
- 检查后台访问权限
- 执行安全检查

### Widget_Archive - beforeRender

页面渲染前触发。

**注册：**
```php
TypechoPlugin::factory('Widget_Archive')->beforeRender = [__CLASS__, 'checkIPAccess'];
```

**用途：**
- 检查前台访问权限
- 记录访客日志

## 配置结构

### 插件配置对象

```php
object(stdClass) {
    // 工作模式
    'mode' => 'smart',  // blacklist, whitelist, smart
    
    // 黑名单处理模式
    'blacklistMode' => 'block',  // block, limit
    
    // IP黑名单
    'blacklist' => "203.0.113.10\n198.51.100.0/24",
    
    // IP白名单
    'whitelist' => "192.168.1.0/24\n10.0.0.0/8",
    
    // 后台访问IP白名单
    'adminWhitelist' => "192.168.1.100\n192.168.1.0/24",
    
    // 非白名单IP重定向URL
    'adminRedirectUrl' => 'https://example.com',
    
    // UA白名单
    'uaWhitelist' => "Googlebot\nBaiduspider",
    
    // URL白名单
    'urlWhitelist' => "^/feed\n^/sitemap\\.xml$",
    
    // 访问间隔（秒）
    'accessInterval' => '10',
    
    // 被禁地区
    'blockedRegions' => "美国\n俄罗斯",
    
    // 敏感词列表
    'sensitiveWords' => "admin\nwp-admin",
    
    // 安全防护开关
    'enableSQLProtection' => ['1'],
    'enableXSSProtection' => ['1'],
    'enableCSRFProtection' => ['1'],
    
    // 自定义拦截提示
    'customMessage' => '抱歉，您的访问被系统安全策略拦截。',
    
    // 调试模式
    'debugMode' => '0',  // 0=关闭, 1=开启
    
    // 登录验证码
    'enableLoginCaptcha' => ['1'],
    
    // 访客日志
    'enableVisitorLog' => ['1'],
    'logRetentionDays' => '30',
    
    // 机器人关键词
    'botKeywords' => "baidu=>百度\ngoogle=>谷歌",
    
    // 完整卸载
    'completeUninstall' => []
}
```

## 数据库表

### typecho_blockip_access_log

拦截日志表。

**结构：**
```sql
CREATE TABLE typecho_blockip_access_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    ip VARCHAR(45) NOT NULL,
    url TEXT,
    reason VARCHAR(255),
    user_agent TEXT,
    location VARCHAR(255),
    block_time INT NOT NULL,
    INDEX idx_ip (ip),
    INDEX idx_block_time (block_time)
);
```

### typecho_blockip_visitor_log

访客日志表。

**结构：**
```sql
CREATE TABLE typecho_blockip_visitor_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    ip VARCHAR(45) NOT NULL,
    url TEXT,
    user_agent TEXT,
    location VARCHAR(255),
    visit_time INT NOT NULL,
    INDEX idx_ip (ip),
    INDEX idx_visit_time (visit_time)
);
```

### typecho_blockip_auto_blacklist_log

自动拉黑日志表。

**结构：**
```sql
CREATE TABLE typecho_blockip_auto_blacklist_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    ip VARCHAR(45) NOT NULL,
    reason VARCHAR(255),
    blacklist_time INT NOT NULL,
    INDEX idx_ip (ip),
    INDEX idx_blacklist_time (blacklist_time)
);
```

## 使用示例

### 示例1：自定义IP检查

```php
// 获取客户端IP
$request = new Request();
$clientIP = IPAccessControl::getRealClientIP($request);

// 获取插件配置
$config = Widget::widget('Widget_Options')->plugin('BlockIPForTypecho');

// 检查是否在黑名单中
if (IPAccessControl::isBlacklisted($clientIP, $config)) {
    BlockHandler::blockAccess($clientIP, 'blacklisted', $config);
}

// 检查是否在白名单中
if (IPAccessControl::isWhitelisted($clientIP, $config)) {
    // 允许访问
    return;
}
```

### 示例2：自定义拦截逻辑

```php
// 自定义检查
$suspicious = false;

// 检查访问频率
if (IPAccessControl::isAccessTooFrequent($clientIP, $config)) {
    $suspicious = true;
}

// 检查地理位置
if (GeoLocation::isBlockedRegion($clientIP, $config)) {
    $suspicious = true;
}

// 拦截可疑访问
if ($suspicious) {
    BlockHandler::blockAccess($clientIP, 'suspicious_activity', $config);
}
```

### 示例3：记录自定义日志

```php
// 记录拦截日志
Logger::logBlockedAccess($clientIP, '自定义拦截原因');

// 记录访客日志
Logger::logVisitorAccess($clientIP, $requestUrl);

// 自动拉黑
IPAccessControl::addToBlacklist($clientIP, $config, '多次攻击尝试');
```

## 扩展开发

### 创建自定义检测器

```php
class CustomDetector
{
    public static function detect(): bool
    {
        // 自定义检测逻辑
        return false;
    }
}

// 在Plugin.php中集成
if (CustomDetector::detect()) {
    BlockHandler::blockAccess($clientIP, 'custom_detection', $config);
}
```

### 添加自定义钩子

```php
// 在activate()中注册
TypechoPlugin::factory('custom/hook')->action = [__CLASS__, 'customAction'];

// 实现方法
public static function customAction()
{
    // 自定义逻辑
}
```

## 版本兼容性

### Typecho 1.2+
所有功能完全支持。

### Typecho 1.1
部分功能可能需要调整。

### PHP 7.4+
推荐使用PHP 7.4或更高版本。

### PHP 8.0+
完全兼容PHP 8.0+。

## 性能指标

### 检查开销
- 前台访问：0ms（不检查）
- 后台访问：1-2ms
- IP匹配：<1ms
- 地理位置查询：<1ms

### 内存占用
- 基础：<1MB
- 带日志：<5MB

### 数据库查询
- 配置读取：1次（缓存）
- 日志记录：1次/访问
- IP检查：0次（内存操作）

## 下一步

了解API后，建议查看：
- [配置指南](./configuration.md)
- [安全最佳实践](./security-best-practices.md)
- [故障排查](./troubleshooting.md)
