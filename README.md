# sql

# 查询每天最后的一条数据sql

select *, from celebrityHistoryDouyin t1 WHERE t1.id = (SELECT max(id) FROM celebrityHistoryDouyin t2 where LEFT(FROM_UNIXTIME(t1.time/ 1000, "%Y-%m-%d %H:%i:%s"),10) = LEFT(FROM_UNIXTIME(t2.time/ 1000, "%Y-%m-%d %H:%i:%s"),10) and celebrityId = 1);