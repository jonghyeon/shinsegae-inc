# hive setting

###### mysql 설치 / hive 유저생성 및 권한 설정 / hive가 사용할 metastore database생성
```
$mysql –u root –p
mysql> grant all privileges on *.* to hive@localhost identified by 'hive' with grant option;
mysql> flush privileges;
mysql> grant all privileges on *.* to hive@'%' identified by 'hive' with grant option;
mysql> flush privileges;`
```

###### 환경변수 등록 hadoop 에 tmp 디렉토리 생성
```
$vi ~/.bash_profile
export HIVE_HOME=/home/hadoop/hive
export PATH=$PATH:$HADOOP_HOME/bin:$HIVE_HOME/bin

$hadoop fs -mkdir /tmp
$hadoop fs -chmod g+w /tmp
```

###### $HIVE_HOME/conf/hive-site.xml
```
<configuration>
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://hadoop01/hive?createDatabaseIfNotExist=true</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.jdbc.Driver</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>hive</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>hive</value>
	</property>
	<property>
		<name>hive.metastore.uris</name>
		<value>thrift://hadoop01:9083</value>
	</property>
</configuration>
```
###### metastore 와 hive 서버 실행
```
$ hive --service metastore
$ hive --service hiveserver2
```

###### hive 접속
```
$beeline
beeline>!connect jdbc:hive2://hadoop01:10000
```
###### 항공지연 데이터 table 생성
```
beeline> CREATE TABLE airline_delay(
Year INT,
Month INT,
DayofMonth INT,
DayOfWeek INT,
DepTime INT,
CRSDepTime INT,
ArrTime INT,
CRSArrTime INT,
UniqueCarrier STRING,
FlightNum INT,
TailNum STRING,
ActualElapsedTime INT,
CRSElapsedTime INT,
AirTime INT,
ArrDelay INT,
DepDelay INT,
Origin STRING,
Dest STRING,
Distance INT,
Taxiln INT,
TaxiOut INT,
Cancelled INT,
CancellationCode STRING COMMENT 'A = carrier,B=weather,C=NAS,D=security',
Diverted INT COMMENT '1=yes,0=no',
CarrierDelay STRING,
weatherDelay STRING,
NASDelay STRING,
SecurityDelay STRING,
LateAircraftDelay STRING)
COMMENT 'The data cons i sts of flight arrival and departure details for all commercial flights within the USA, from October 1987 to April 2008'
PARTITIONED BY (delayYear INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;
```

###### CSV 파일의 첫줄을 제거 함
```
$ sed -e '1d' 1991.csv > 1991_new.csv
```

###### 첫줄 제거한  CSV 파일을 hive airline_delay 테이블에 로딩
```
beeline> LOAD DATA LOCAL INPATH '/home/hadoop/input/1991_new.csv'
OVERWRITE INTO TABLE airline_delay
PARTITION (delayYear='1991');
```

###### hive 에서 테이블 확인과 테이블 정보 확인
```
beeline> show tables;
beeline> desc airline_delay;
```


###### 1991년도의 지연건수 query
```
beeline> SELECT COUNT(1) FROM airline_delay WHERE delayYear = 1991
```


###### 1991년도의 도착 지연 건수 query
```
beeline> SELECT Year, Month, COUNT(*) AS arrive_delay_count FROM airline_delay WHERE delayYear = 1991 AND ArrDelay > 0 GROUP BY Year, Month;
```


###### 1991년도의 평균 지연시간 연도와 월별로 계산 query
```
beeline> SELECT Year, Month, AVG(ArrDelay) AS arrive_delay_time, AVG(DepDelay) AS avg_departure_delay_time
FROM airline_delay WHERE delayYear = 1991 AND ArrDelay > 0 GROUP BY Year, Month;
```
###### 항공사 정보 데이터 로딩을 위헤 데이터의 " 를 제거
```
$ find . -name carriers.csv -exec perl -p -i -e 's/"//g' {} \;
```
###### 항공사 정보 데이터 테이블 생성
```
beeline> CREATE TABLE carrier_code(Code STRING, Description STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;
```

###### 항공사 정보 데이터를 carrier_code teable 에 로딩
```
beeline> LOAD DATA LOCAL INPATH '/home/hadoop/input/carriers.csv'
OVERWRITE INTO TABLE carrier_code;
```
###### query
```
beeline> SELECT A.Year, A. UniqueCarrier, B.Description, COUNT(*)
FROM airline_delay A
JOIN carrier_code B ON (A.UniqueCarrier = B.Code)
WHERE A.ArrDelay > 0
GROUP BY A.Year, A.UniqueCarrier, B.Description;
```
###### 공항 정보 데이터 로딩을 위해 데이터의 " 를 제거
```
$ find . -name airports.csv -exec perl -p -i -e 's/"//g' {} \;
```

###### 공항 정보 데이터 테이블 생성
```
beeline> CREATE TABLE airport_code(Iata STRING,Airport STRING,City STRING,
State STRING,Country STRING,Lat Double,Longitude Double)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
STORED AS TEXTFILE;
```
###### 공항 정보 데이터를  airport_code 로딩
```
beeline> LOAD DATA LOCAL INPATH '/home/hadoop/input/airports.csv'
OVERWRITE INTO TABLE airport_code;
```
###### 출발 공항 코드(Origin)와 도착 공항 코드(Dest) ，airport_code 테이블을 조인해 공항별 지연 건수를 계산하는 쿼리
```
beeline> SELECT A.Year, A.Origin, B.Airport, A.Dest, C.Airport, COUNT(*)
FROM airline_delay A
JOIN airport_code B ON (A.Origin = B.Iata)
JOIN airport_code C ON (A.Dest = C.Iata)
WHERE A.ArrDelay > 0
GROUP BY A.Year, A.Origin, B.Airport, A.Dest, C.Airport;
```