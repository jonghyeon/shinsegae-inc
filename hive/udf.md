##UDF
###
```
0: jdbc:hive2://hadoop01:10000> show functions;
```
######함수의 정의를 보려면 describe 사용
```
0: jdbc:hive2://hadoop01:10000> describe function abs;
```
######UDF 등록
```
0: jdbc:hive2://hadoop01:10000> ADD JAR /home/hadoop/share/Fullname.jar;
0: jdbc:hive2://hadoop01:10000> CREATE TEMPORARY FUNCTION fullname AS 'hive.udf.sample.Fullname';
```
######UDF 사용
```
0: jdbc:hive2://hadoop01:10000>describe function fullname;
0: jdbc:hive2://hadoop01:10000>DESCRIBE FUNCTION EXTENDED fullname;
0: jdbc:hive2://hadoop01:10000>SELECT ymd,fullname(symbol) FROM Stocks where symbol='AAPL';
```
