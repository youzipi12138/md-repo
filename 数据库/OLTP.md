# 一、OLTP 到底是什么？

OLTP 全称：

Online Transaction Processing，在线事务处理。

拆开理解：

- Online：在线，实时响应业务请求。
    
- Transaction：事务，一组需要保证一致性的数据库操作。
    
- Processing：处理。
    

它的核心目标是：

快速、可靠地处理大量日常业务读写请求。

举个例子：

假设你的达人营销系统有一张达人表：

`talent`

|id|name|fans_count|
|---|---|---|
|1|张三|100000|
|2|李四|200000|
|3|王五|50000|

当用户修改张三的粉丝数量：

```
UPDATE talent
SET fans_count = 120000
WHERE id = 1;
```

这就是一个典型的 OLTP 操作。

因为它关注的是：修改是否成功、数据是否准确、能否快速返回结果。

# 二、OLTP 有哪些典型特点？

主要有以下 6 个特点：

|特点|解释|
|---|---|
|高并发|很多用户同时操作数据库|
|实时性|请求通常需要快速响应|
|频繁读写|INSERT、UPDATE、DELETE、SELECT 都很多|
|小事务|一次操作通常涉及少量数据|
|强一致性|需要保证事务正确执行|
|数据量相对较小|单次查询通常只涉及少量记录|

这里需要注意：

OLTP 数据库本身也可以存储非常大的数据量，只是它的查询和事务设计更偏向于高频、小范围的业务操作。

# 三、哪些数据库属于 OLTP？

常见的 OLTP 数据库：

[Kanbel Technologies](https://images.openai.com/static-rsc-4/lIUH81tejlkswCXyNrWuAAxHsjybAQlaIuDKv-Z_XUmCzHcKcT0u90kraHfiEhBc5_c_lgT8bKGRksbnHzwIig_sBovMW8wilv9VZzv32K_9sTK0jrXJGhpYyCvO-HD-GtYa78cFp6aregV3pNY-Eylq1Zne85ayQKZjtQ3hhl4?purpose=inline)

MySQL

最常见的关系型业务数据库，适合用户、订单、达人等业务数据。

[Postgresql Logo PNG Transparent With Clear Background ID 473068 | TOPpng](https://images.openai.com/static-rsc-4/x6sDZ8oKI9URixQ5E82vqCQV1aR38qK7E8n0HKggx6_cD8G7QtlPuHlyO99GalHPJOycjs41bPgfrRTqdHFqayPWJl57hxNttQynREzHXSUSsui_zVAB_ssyc1xdm_msJ5GiDzf23C9Z2RuuoyVCc6J0AbuAqX5Aul_x1J78q2I?purpose=inline)

PostgreSQL

功能丰富，支持复杂 SQL、事务和多种数据类型。

[Oracle DataBase Enterprise Edition](https://images.openai.com/static-rsc-4/MAD_OixjnJKShdjBwH7Cp_XBcVd2j47Gw0_EtlJryqE9bcUaqDrYAwrjv2SgvHvJilu_FKsRQ-ZXFFoSFMRsNM4uReG09EtoHSCv82PlDCPtncI-SrNxjM3b6prqaYdYM5FzKVfMryf88wZit5yU137XTlGEN8Wvl2hEDVDpCY0?purpose=inline)

Oracle

传统企业级数据库，常用于大型核心业务系统。

[TiDB Logo PNG, Vector (AI, EPS, CDR, PDF, SVG) - iconLogoVector](https://images.openai.com/static-rsc-4/snDRJSWsfaeAos5B6oTC8K64-lKZ1dSxG4NsWvJJjUFlXBACgSGgdzohWa9pP0o5V99I0AP9Xfh5uFaXOO4Ba6MlgxvW7cFKqX9jlltavqP6Kw-U04hWUKM8287smmKg7SMRwmkMcPsWqrsjjZT2CHl59bp3mrh6xkxSeyx7mRY?purpose=inline)

TiDB

分布式关系型数据库，兼容 MySQL 协议，支持水平扩展。
# 五、OLTP 为什么强调 ACID？

OLTP 最重要的技术基础之一就是数据库事务 ACID。

|特性|含义|举例|
|---|---|---|
|A：原子性|要么全部成功，要么全部失败|转账不能只扣款不入账|
|C：一致性|数据必须满足业务约束|账户余额不能违反业务规则|
|I：隔离性|并发事务之间互不产生错误干扰|两个人同时修改一条记录|
|D：持久性|提交成功后数据不能轻易丢失|数据库重启后数据仍然存在|

这也是为什么 MySQL InnoDB 会提供事务、锁、MVCC、Redo Log、Undo Log 等机制。