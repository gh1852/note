# MySQL 日期计算指南

## 1. 3天后

### 保留当前时间
```sql
-- 推荐：DATE_ADD
SELECT DATE_ADD(NOW(), INTERVAL 3 DAY) as three_days_later;

-- 等价写法：直接运算
SELECT NOW() + INTERVAL 3 DAY as three_days_later;

-- 简写形式
SELECT ADDDATE(NOW(), 3) as three_days_later;
```

### 从当天 23:59:59 开始
```sql
SELECT TIMESTAMP(DATE_ADD(CURDATE(), INTERVAL 3 DAY), '23:59:59') as target_time;
```

---

## 2. 3天前的23:59:59

```sql
SELECT TIMESTAMP(DATE_SUB(CURDATE(), INTERVAL 3 DAY), '23:59:59') as three_days_ago_235959;
```

---

## 核心函数

| 函数/操作符 | 说明 |
|-------------|------|
| `DATE_ADD()` | 日期加法 |
| `DATE_SUB()` | 日期减法 |
| `NOW()` | 当前日期时间 (YYYY-MM-DD HH:MM:SS) |
| `CURDATE()` | 当前日期 (YYYY-MM-DD，不含时间) |
| `TIMESTAMP(date, time)` | 组合日期和时间 |
| `INTERVAL` | 时间间隔关键字 |
| `+ / -` | 日期直接加减法 |

---

## 时间单位

| 单位 | 示例 |
|------|------|
| DAY | `INTERVAL 3 DAY` |
| HOUR | `INTERVAL 5 HOUR` |
| MINUTE | `INTERVAL 30 MINUTE` |
| SECOND | `INTERVAL 45 SECOND` |
| MONTH | `INTERVAL 1 MONTH` |
| YEAR | `INTERVAL 1 YEAR` |

---

## 常用组合模式

### 未来时间
```sql
-- 从当前时间算起
DATE_ADD(NOW(), INTERVAL N 单位)

-- 从当天开始算起
TIMESTAMP(DATE_ADD(CURDATE(), INTERVAL N DAY), 'HH:MM:SS')
```

### 过去时间
```sql
-- 从当前时间算起
DATE_SUB(NOW(), INTERVAL N 单位)

-- 从当天开始算起
TIMESTAMP(DATE_SUB(CURDATE(), INTERVAL N DAY), 'HH:MM:SS')
```

---

## 示例场景

### 查询未来3天的订单
```sql
SELECT * FROM orders
WHERE order_date >= DATE_ADD(NOW(), INTERVAL 3 DAY);
```

### 设置用户30天过期
```sql
UPDATE users
SET expire_date = DATE_ADD(created_at, INTERVAL 30 DAY);
```

### 统计本月数据
```sql
SELECT * FROM records
WHERE created_at >= DATE_ADD(CURDATE(), INTERVAL -1 MONTH);
```

---

## 注意事项

- `NOW()` 返回完整的时间戳；`CURDATE()` 只返回日期部分
- 使用 `CURDATE()` 可确保从 00:00:00 开始计算
- 复杂时间计算建议使用 `TIMESTAMP()` 组合日期和时间
- `INTERVAL` 语法是 MySQL 特有标准，可读性好
