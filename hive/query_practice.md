###### 테이블 생성
```
0: jdbc:hive2://hadoop01:10000>CREATE TABLE IF NOT EXISTS stocks(`exchange` STRING,
symbol STRING,
ymd STRING,
price_open FLOAT,
price_high FLOAT,
price_low FLOAT,
price_close FLOAT,
volume INT,
price_adj_close FLOAT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
0: jdbc:hive2://hadoop01:10000>desc stocks ;
```
###### 데이터 업로드
```
0: jdbc:hive2://hadoop01:10000>
load data local inpath '/home/hadoop/sample_data/NASDAQ/NASDAQ_daily_price*' overwrite into table stocks;
stocks;0: jdbc:hive2://hadoop01:10000>
load data local inpath '/home/hadoop/sample_data/NYSE/NYSE_daily_prices*' into table stocks;
```
######데이터 갯수 확인
```
0: jdbc:hive2://hadoop01:10000>SELECT count(*) FROM stocks;
```
######뉴욕 거래소 및 나스닥 전체 업종 수
```
0: jdbc:hive2://hadoop01:10000>SELECT count(DISTINCT symbol) FROM stocks;
```
######애플의 각 연도별 종가 평균
```
0: jdbc:hive2://hadoop01:10000>SELECT year(ymd), avg(price_close) FROM stocks WHERE `exchange` = 'NASDAQ' AND symbol = 'AAPL‘ GROUP BY year(ymd);
```
######2010 장종가 평균 상위 5개 업체
```
0: jdbc:hive2://hadoop01:10000>select symbol, avg(price_open) as openprice from stocks where year(ymd) = '2010' group by symbol sort by openprice DESC limit 5;
```
######View Table
```
0: jdbc:hive2://hadoop01:10000>SELECT s2.year, s2.avg FROM (SELECT year(ymd) AS year, avg(price_close) AS avg FROM stocks WHERE `exchange` = 'NASDAQ' AND symbol = 'AAPL' GROUP BY year(ymd)) s2 WHERE s2.avg > 50.0;
0: jdbc:hive2://hadoop01:10000>create view stockview AS SELECT year(ymd) AS year, avg(price_close) AS avg FROM stocks WHERE `exchange` = 'NASDAQ' AND symbol = 'AAPL' GROUP BY year(ymd);
0: jdbc:hive2://hadoop01:10000>select stockview.year, stockview.avg from stockview where stockview.avg > 50;
```
###### partition 테이블 만들기
```
0: jdbc:hive2://hadoop01:10000>CREATE TABLE IF NOT EXISTS stocks_part (
ymd STRING,
price_open FLOAT,
price_high FLOAT,
price_low FLOAT,
price_close FLOAT,
volume INT,
price_adj_close FLOAT)
PARTITIONED by ( exchange STRING, symbol STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
```
######partition 테이블에 데이터 로드
```
0: jdbc:hive2://hadoop01:10000>INSERT OVERWRITE TABLE stocks_part PARTITION(`exchange`='NASDAQ', symbol ='AAPL') SELECT ymd,price_open,price_high,price_low,price_close,volume int,price_adj_close
FROM stocks WHERE `exchange` = 'NASDAQ' and symbol='AAPL';
```
######확인
```
0: jdbc:hive2://hadoop01:10000>show partitions stocks_part;
```
######다이니믹 Partition
######설정(기본값)
```
hive.exec.dynamic.partition.mode=strict;
hive.exec.dynamic.partition = false;
hive.exec.max.dynamic.partitions.pernode=100;
hive.exec.max.dynamic.partitions=1000;
hive.exec.max.created.files=100000;
```
######partition 테이블에 데이터 로드
```
0: jdbc:hive2://hadoop01:10000>INSERT OVERWRITE TABLE stocks_part PARTITION(`exchange`, symbol) SELECT ymd,price_open,price_high,price_low,price_close,volume int,price_adj_close,`exchange`,symbol
FROM stocks;
```
######table에 색인 생성
######symbol컬럼에 색인 생성
```
0: jdbc:hive2://hadoop01:10000>CREATE INDEX simple_index ON TABLE stocks (symbol) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD;
```
######index 확인
```
0: jdbc:hive2://hadoop01:10000>SHOW FORMATTED INDEX ON stocks;
```
######query수행
```
0: jdbc:hive2://hadoop01:10000>SELECT s2.year, s2.avg FROM (SELECT year(ymd) AS year, avg(price_close) AS avg FROM stocks WHERE symbol = 'AAPL' GROUP BY year(ymd)) s2 WHERE s2.avg > 50.0;
```
######인덱스 생성
```
0: jdbc:hive2://hadoop01:10000>alter index simple_index on stocks rebuild;
```
######필요한 파일만 다시 로드
```
0: jdbc:hive2://hadoop01:10000>INSERT OVERWRITE DIRECTORY "/tmp/index_result" SELECT `_bucketname` , `_offsets` FROM default__stocks_simple_index__ where symbol='AAPL';
```
######기존 테이블과 연결
```
0: jdbc:hive2://hadoop01:10000>SET hive.index.compact.file=/tmp/index_result;
0: jdbc:hive2://hadoop01:10000>SET hive.input.format=org.apache.hadoop.hive.ql.index.compact.HiveCompactIndexInputFormat;
```
######query수행
```
0: jdbc:hive2://hadoop01:10000>SELECT s2.year, s2.avg FROM (SELECT year(ymd) AS year, avg(price_close) AS avg FROM stocks WHERE symbol = 'AAPL' GROUP BY year(ymd)) s2 WHERE s2.avg > 50.0;
```
