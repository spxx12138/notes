# SQL练习 #

*基于mysql 5.7*

##### 1. 并、交、差 #####

使用交并差运算时，必须保证：
- 列的顺序和数量必须相同。
- 相应列的数据类型必须兼容或可转换。

**并集（UNION）：** 使用UNION关键字，加all关键字则查询结果不去重。

```sql
select sid from student union select sid from sc;
select sid from student union all select sid from sc;
```

**交集（INTERSECT）：** mysql不支持INTERSECT关键字，不过可以使用inner join或者in达到交集的目的。
- inner join
```sql
select distinct student.sid from student INNER JOIN sc on sc.sid = student.sid ;
select distinct student.sid from student INNER JOIN sc using(sid);
```
使用using可以代替on，上面两条sql语句执行结果相同。
- in
```sql
select distinct sid from student where sid in (select sid from sc)
```

**差集（MINUS）：** mysql不支持MINUS关键字，不过可以使用left join或者not in达到差集的目的。
- left join
```sql
select student.sid,sc.sid from student LEFT JOIN sc using(sid) where sc.sid is null
```
- not in
```sql
select distinct sid from student where sid not in (select sid from sc)
```