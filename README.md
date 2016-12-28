# logstashSucks
曾经走过的坑

####1、logstash在单条日志量长度过大时，grok效率低，耗费CPU
INFO 2016-12-28 08:00:00 - user admin login    
INFO 2016-12-28 08:00:00 - jso output {......} size more than 7k    
想要直接从这样的日志中提取出用户admin 字段用于kibana出用户登录统计，目前没有好的办法，请不吝赐教,论坛建议使用dissect，空了试下
####2、logstash jdbc plugin    
我在中国！我在中国！我在中国！    
	statement => "SELECT t.times,t.username,t.password FROM dgweb.dbo.test t where dateadd(hour,-8,t.times) >= :sql_last_value
####3、
