

#### LAG 和 LEAD 函数

**LAG**

LAG(col,n,DEFAULT) 用于统计窗口内往上第n行值

参数1为列名，参数2为往上第n行（可选，默认为1），参数3为默认值（当往上第n行为NULL时候，取默认值，如不指定，则为NULL）

**LEAD**

与LAG相反

LEAD(col,n,DEFAULT) 用于统计窗口内往下第n行值

参数1为列名，参数2为往下第n行（可选，默认为1），参数3为默认值（当往下第n行为NULL时候，取默认值，如不指定，则为NULL）

[**Hive 分析函数lead、lag实例应用**](https://blog.csdn.net/kent7306/article/details/50441967)




### 关联表（join）

在hive中，关联有4种方式：

内关联：join on

左外关联：left join on

右外关联：right join on

全外关联：full join on  

###  Hive 没有update 方法， 只能insert 

###  清除表中数据
ALTER TABLE db.table  DROP PARTITION (dt='2019-12-31');

--添加多个字段
alter table bron_lpss_lpss_order_info_cur add columns
(
    order_source string comment '订单来源',
    mid string comment '新会员id',
    bank_name string comment '银行行名'
);
