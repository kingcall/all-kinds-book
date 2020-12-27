改变日志格式进行调试
hive --hiveconf hive.root.logger=DEBUG,console

内部转外部
alter table tableA set TBLPROPERTIES('EXTERNAL'='true')

外部转内部
alter table tableA set TBLPROPERTIES('EXTERNAL'='false')


修改表结构
	ALTER TABLE employee CHANGE name ename String;
	ALTER TABLE employee RENAME TO emp;
	alter table fct_user_ctag_today CHANGE COLUMN rpt_tag rpt_tag int comment '1新客，2新转老，3新注册，5老客';
	alter table add partition(dt='20190309') location ''——内部表外部表都支持
	对于内部表，我们可以直接在其目录下创建分区文件夹，然后使用 msck  repair table test3——适用于一次添加多个分区

删除分区
ALTER TABLE dws.realtime_large_screen DROP IF EXISTS PARTITION (dt='2018-12-16');




查看表描述
describe formatted src_workorder_type;
	Database:           	src                 	 
	Owner:              	xuhong              	 
	CreateTime:         	Fri Nov 16 18:33:44 CST 2018	 
	LastAccessTime:     	UNKNOWN             	 
	Retention:          	0                   	 
	Location:           	hdfs://emr-cluster/user/hive/warehouse/src.db/src_workorder_type	 
	Table Type:         	MANAGED_TABLE       	 
	Table Parameters:	 	 
		comment             	工单类型记录表             
		numFiles            	203                 
		numPartitions       	113                 
		numRows             	0                   
		rawDataSize         	0                   
		totalSize           	877831              
transient_lastDdlTime	1542364424    