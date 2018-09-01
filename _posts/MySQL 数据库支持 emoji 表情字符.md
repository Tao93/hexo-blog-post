---
title: MySQL 数据库支持 emoji 表情字符
tags: [MySQL]
---

兴冲冲的整个博客，死皮赖脸的叫几个同学来评论，结果评论中输入的 emoji 表情最后会变成问号，试想别人评论「写的真棒 😄」结果变成了「写的真棒 ？」，这是何其的尴尬。

继续回到这个问题本身。我登录到数据库一查，发现数据库中存的评论文本就是个问号，这说明 emoji 表情存到数据库的过程出问题了。赶紧一查，发现网上还是很容易找到了解决方法。问题的原因是 MySQL 存储文本时默认的 UTF-8 仅支持 3 个字节编码，而 emoji 是 4 字节编码的，所以存储过程出问题了。所以需要将 charset 设置为 UTF-8 的超集 UTF-8mb4，虽然我也不知道这个 UTF-8mb4 是何方神圣。

首先，需要在配置文件 my.cnf (在我的机器上这个文件路径是 /etc/my.cnf) 中添加一下内容：

```
[client]
default-character-set=utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect=’SET NAMES utf8mb4'

[mysql]
default-character-set=utf8mb4
```

然后，需要修改已有库、表、字段的 charset。

```sql
# 修改库的 
ALTER DATABASE <database_name> CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci; 

# 修改表的 
ALTER TABLE <table_name> CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci; 

# 修改字段的 
ALTER TABLE <table_name> CHANGE <column_name> <column_name> <original_column_type> CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

最后重新启动 MySQL 服务：

```bash
sudo /etc/init.d/mysql stop sudo /etc/init.d/mysql start
```

另外，mysqldump 时，也需要进行额外指定 char set：

```bash
mysqldump -default-character-set-utf8mb4 -u <db_user_name> -p --databases <db_name> --lock-all-tables > <file_name>
```
