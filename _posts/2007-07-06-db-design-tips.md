---
title: 几个数据库设计技巧
---

## 减少字段
将业务对象或者部分属性序列化后保存在一个字段中。如果对象较小或者属性比较琐碎，不想一一建立字段来保存。可以将其序列化成xml保存在一个字符型的字段中。

## 系统信息表
建立系统信息表。用于记录数据库版本，以及相关配置信息、元数据等。配置信息特别是部署中需要设置的最好放在数据库中，如果以文件的形式(如XML)配置，在升级的时候会比较麻烦。如增加了配置项就需要覆盖旧文件，并重新配置。

## 树形结构
自身存在树形关系的表建立FullPath字段和Level字段，记录完整路径和所属层次。  

| ID | ParentId | FullPath | Level | Name |
| -- | -------- | -------- | ----- | ---- |
| 1 | 0 | 0	    | 0 | 祖宗 |
| 2 | 1 | 1	    | 1 | 爷爷 |
| 3 | 2 | 1.2   | 2 | 老子 |
| 4 | 2 | 1.2   | 2 | 老子的兄弟 |
| 5 | 3 | 1.2.3 | 3 | 自己 |

找祖宗所有后代：select * from t where FullPaht like '1.%'  
找爷爷的所有下一代：select * from t where ParentId = 2  
找自己所有的上一辈：select * from t where level= 2  

## 主从表
在主表中增加统计字段，避免查询时进行count、sum计算。如果需要频繁删改从表，则不建议使用，否则导致更新主表麻烦。

## 乐观锁
避免多人同时操作，后提交的信息将先提交的信息覆盖。增加version字段，每次更新时，先检查数据库中的version值是否大于当前值。如果是，证明数据库中的记录已发生了变化，此时更新将覆盖变化。如果不是，更新是不会影响其他人的操作，提交信息同时将version值加1。