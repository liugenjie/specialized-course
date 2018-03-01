# 数据库学习笔记 - 结构化查询语言SQL

SQL 是 Structured Query Language 的缩写，是数据库的标准主流语言。

SQL 的语言特点是集数据定义、数据查询、数据操纵、数据控制功能于一体，综合、通用、功能强。

使用 SQL 有两种方式，一种是联机问答形式，一种是嵌入某种高级程序设计语言的程序中。

## 数据定义

- 建表

  CREATE TABLE <表名>
  (
    <列名> <数据类型> <列级完整性约束条件>,
    <列名> <数据类型> <列级完整性约束条件>,
    <列名> <数据类型> <列级完整性约束条件>,
    <表记完整性约束条件>
  )

  <列级完整性约束条件> 包括：

  PRIMARY KEY 主键约束
  UNIQUE 唯一性约束
  CHECK 列取值必须符合所设置的条件
  DEFAULT 列定义默认值
  NOT NULL 指定是否允许空值
  FOREIGN KEY 设置外键

  如果完整性约束条件涉及该表的多个属性列（如复合属性组成的主键），应定义在表级上，<表级完整性约束条件> 包括：

  PRIMARY KEY
  CHECK
  FOREIGN KEY

  举例建表：

  ```sql
  CREATE TABLE 学生
  (
    学号 VARCAHR(20) PRIMARY KRY,
    姓名 VARCHAR(20) NOT NULL,
    性别 VARCHAR(20) DEFAULT '男',
    出生日期 SMALLDATETIME,
    身份证号 VARCHAR(20) UNIQUE
  )
  GO
  ```

- 表结构的修改

  ```sql
  ALTER TABLE <表名>
    ADD <列名> <数据类型> <列级完整性约束>
    ALTER COLUMN <列名> <数据类型> <列级完整性约束>
    DROP <完整性约束名>
    DROP COLUMN <列名>
  ADD子句用于增加新列和新的完整性约束条件。
  ALTER COLUMN用于修改列名、数据类型、列级完整性约束条件。
  DROP子句用于删除指定的完整性约束条件。
  DROP COLUMN子句用于删除指定列。
  ```

- 基本表删除

  DROP TABLE <表名>

## 索引

索引是为了加速表中数据行的检索而创建的一种分散数据结构。

- 建立索引

  在表上建立索引：CREATE [索引类别] INDEX <索引名> ON <表名>

  在列上建立索引：CREATE [索引类别] INDEX <索引名> ON <表名>(<列名>[ASC|DESC])
  索引类别为可选项，包括：UNIQUE、CLUSTERED、NONCLUSTERED。
  ASC表明索引按升序排列，DESC表明索引按降序排列。

- 删除索引

  DROP INDEX <索引名>

## 数据查询

对数据的查询是数据库最核心的操作。基本查询语句形式为SELECT-FROM-WHERE。

- 投影查询

  最基本的 SELECT 语句只有两部分，分别要返回的列和这些列源于的表。这叫做投影查询。例如：SELECT <目标列表达式> FROM <表名或视图名>

- 条件查询

  查询课程号为『c1』，60-100分的学生的姓名和成绩
  SELECT 姓名, 成绩 FROM 选课 WHERE 课程号 = 'c1' AND(成绩 BETWEEN 60 AND 100)

  1. 部分匹配查询
    不知道完全精确的值时，用户可以用LIKE或NOT LIKE进行部分匹配查询（模糊查询）。用『%』做多个字符通配符，『_』做一个字符通配符。
    查询计算机学院所有姓赵的学生：SELECT 姓名 FROM 学生 WHERE 姓名 LIKE '赵%' AND(所在院系 = '计算机')
  2. 空值查询
    某个字段没有值称为具有空值。
    查询所有没填写家庭住址的同学班级和姓名 SELECT 班级, 姓名 FROM 学生 WHERE 家庭住址 IS NULL
  3. 限定结果返回行数
    用TOP关键字表明查看前几项，TOP后跟常数或表达式，可选加PERCENT表示百分比。
    查看学生表前百分之20的同学姓名 SELECT TOP 20 PERCENT FROM 学生

- 连接查询

通过连接操作，将多个表连接起来，从多个表中查询数据。

1. 交叉连接（自然连接）查询
  相当于两个基本表的笛卡儿积。交叉连接使用关键字 CROSS JOIN。
  查询所有学生选修所有课程的信息 SELECT * FROM 学生 CROSS JOIN 课程
2. 内连接查询
  通过比较运算符进行基本表内某些列数据的比较操作。
  查询计算机学院分数高于90的所有学生姓名及成绩 SELECT 学生.姓名, 选课.成绩 FROM 学生, 选课 WHERE(学生.学号 = 选课.学号) AND(成绩 > 90)
  如果属性是唯一的，那么可以省略表名前缀。

- 排序

需要对结果进行排序，在SELECT语句中使用ORDER BY子句，后跟列名，升序ASC，降序DESC，默认升序。
查询选修c1课的同学姓名和成绩，并按成绩降序排列 SELECT 姓名, 成绩 FROM 选课 WHERE 课程号 = 'c1' ORDER BY 成绩 DESC

- 分组及计算查询

有时需要对基本表中数据按照一定条件进行分组汇总或求平均值，就要在查询语句中与GROUP BY子句一起使用。COUNT、SUM、AVG、MAX、MIN。
查询各院系学生人数 SELECT 所在院系 AS 学院, COUNT (*) AS 人数 FROM 学生 GROUP BY 所在院系
在分组查询时，需要分组满足某个条件才检索，此时用HAVING子句限定分组。HAVING跟在GROUP BY之后，不能单独使用。

- 子查询

  有时候需要在其他SELECT语句的查询结果之上在进行查询，这里就用到子查询，也称嵌套查询。

  比如查询出平均成绩高于80分的学生学号和姓名
  SELECT 学号, 姓名 FROM 学生 WHERE 学号 IN (
    SELECT 学号 FROM 选课 GROUP BY 学号 HAVING AVG(成绩) > 80
  )

  使用量词查询，查询出生科院平均成绩高于计科院最低成绩同学的学生学号和姓名
  SELECT 学号, 姓名 FROM 学生 WHERE 学号 IN (
    SELECT 学号 FROM 选课 WHERE 成绩 > ANY (
      SELECT 成绩 FROM 选课 WHERE 学号 IN (
        SELECT 学号 FROM 学生 WHERE 学院 = '计算机科学与技术学院'
      )
    )
  ) AND 学院 = '生命科学学院'

- 集合操作

  包含交、并、差。只需要在两段SELECT语句之间使用交UNION、并INTERSECT、差EXCEPT即可。

## 数据操作

数据操作功能用来插入INSERT、更新UPDATE、删除DELETE数据库数据的功能。

- 插入数据

  插入数据语句用于向数据库表或视图中加入一行数据。
  向学生表插入我的信息 INSERT INTO 学生 VALUES('2013115010148', '孙恺', '男')

- 更新数据

  更新数据需要用UPDATE语句与SET子句。
  更新一个元组的值 UPDATE 学生 SET 学院 = '计科' WHERE 学号 = '2013115010148'
  UPDATE 选课 SET 成绩 = 0 WHERE '学号' IN (
    SELECT 学号 FROM 选课 WHERE 课程号 = 'c0'
  ) AND 学院 = '计科'

- 删除数据

  根据学号删除一个学生 DELETE FROM 学生 WHERE 学号 = '2013115010148'

- 保持数据一致性

  关于一名学生的各种数据可能存在于不同的表中，那么对一个表格的数据进行了修改（删除），则要考虑是否应该删除其他表格中关于记录的数据。

## 视图

  视图是一张虚表，从一个或几个基本表中导出的表，视图对应的数据并不实际以视图结构存储在数据库中。

  对学生部分信息建立视图 CREATE VIEW 学生家庭信息视图 AS SELECT 姓名, 出生日期, 家庭住址 FROM 学生
  把学生家庭信息视图修改为计算机女学生学号和姓名 ALTER VIEW 学生家庭信息视图 AS SELECT 学号, 姓名 FROM 学生 WHERE 学院 = '计科院' AND 性别 = '女'
  视图的删除 DROP VIEW 学生家庭信息视图
  视图的更新 UPDATE 学生家庭信息视图 SET 姓名 = '李涛' WHERE 学号 = '2013115010148'

## 数据控制

  把查询学生表权限授给用户U1  GRANT SELECT ON 学生 TO U1
  把用户U4修改学号的权限收回 REVOKE UPDATE(学号) ON 学生 FROM U4
  拒绝U2和U3用户对课程表进行修改和删除权限 DENY UPDATE, DELETE ON 课程 TO U2, U3

## Reference

- [数据库学习笔记-03结构化查询语言SQL](http://blog.talisk.cn/blog/2016/01/08/DB-Learning-04-SQL/)