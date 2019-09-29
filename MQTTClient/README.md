# MQTT Client Plugin

## 1. Build(tested on Linux 64bit GCC version 5.4.0)

1. This plugin has been successfully compiled with GCC (version 5.4.0) on 64-bits Linux operating system.
2. Before compiling, install [CMake](https://cmake.org/).
3. Create a 'build' directory. Under the directory, run `cmake ..` and `make` to generate 'libPluginMQTTClient.so'.

```
mkdir build
cd build
cmake ..
make
```

## 2. Load Plugin

Use function `loadPlugin` to load MQTT client plugin.

```
loadPlugin("/YOUR_PATH/MQTTClient/PluginMQTTClient.txt"); 
```
Note: users should define their own path here.

## 3. Publish

### 3.1 Connect to an MQTT server/broker

**Syntax**

```
mqtt::connect(host, port, [QoS=0], [formatter], [batchSize=0])
```
Connect to an MQTT server/broker. It returns a connection object which can be explicitly called to close the `close` function. It is automatically released when the reference count is 0. 

**Arguments**

- 'host' is a string indicating the IP address of an MQTT server/broker.

- 'port' is an integer indicating the port number of an MQTT server/broker.

- 'QoS' is an integer indicating the quality of service. 0: at most once; 1: at least once; 2: only once. It is optional and the default value is 0.

- 'formatter' is a function to package the published data in a format. Currently supported functions are `createJsonFormatter` and `createCsvFormatter`.

- 'batchSize' is an integer. When a table is published in batches, 'batchSize' indicates the number of rows in a batch.

**Example**
```
f=createCsvFormatter([INT, TIMESTAMP, DOUBLE, DOUBLE,DOUBLE], ',', ';' )
conn=connect("test.mosquitto.org",1883,0,f,50)
```

### 3.2 Publish

**Syntax**

```
mqtt::publish(conn, topic, obj)
```

Publish one or more messages to an MQTT server/broker. 

**Arguments**

- 'conn' is the object generated by function `connect`.

- 'topic' is a string indicating the subscription topic.

- 'obj' is the content of the message to be published, which can be a table or a string or an array of strings.


**Example**

```
mqtt::publish(conn,"dolphindb/topic1","welcome")
mqtt::publish(conn,"devStr/sensor001",["hello world","welcome"])
t=table(take(0..99,50) as hardwareId ,take(now(),
		50) as ts,rand(20..41,50) as temperature,
		rand(50,50) as humidity,rand(500..1000,
		50) as voltage)
mqtt::publish(conn,"dolphindb/device",t)		

``` 

### 3.3 Close the connection

**Syntax**

```
mqtt::close(conn)
```
Disconnect from the server/broker.

**Arguments**

- 'conn' is the object returned by function `connect`.

**Example**
```
mqtt::close(conn)
```

## 4. Subscribe/Unsubscribe

### 4.1 Subscribe

**Syntax**

```
mqtt::subscribe(host, port, topic, parser, handler)
```

**Arguments**

- 'host' is a string indicating the IP address of an MQTT server/broker.

- 'port' is an integer indicating the port number of an MQTT server/broker.

- 'topic' is a string indicating the subscription topic.

- 'parser' is a function to parse the subscribed messages. Currently supported functions are `createJsonParser` and `createCsvParser`.

- 'handler' is a function or a table to process the subscribed data.

**Details**

Subscribe to an MQTT server/broker. It returns a connection object which can be explicitly called to close the `close` function. It is automatically released when the reference count is 0. 

**Example**

```
p = createCsvParser([INT, TIMESTAMP, DOUBLE, DOUBLE,DOUBLE], ',', ';' )
sensorInfoTable = table( 10000:0,`deviceID`send_time`temperature`humidity`voltage ,[INT, TIMESTAMP, DOUBLE, DOUBLE,DOUBLE])
 
conn = mqtt::subscribe("192.168.1.201",1883,"sensor/#",p,sensorInfoTable)
```

### 4.2 Unsubscribe

**Syntax**

```
mqtt::unsubcribe(conn)  
```

**Arguments**

- 'conn' is the object returned by function `subscribe`.

**Example**

```
mqtt::unsubcribe(conn)    
```

## 5. Formatter/Parser

### 5.1 createCsvFormatter

**Syntax**

```
mqtt::createCsvFormatter([format], [delimiter=','], [rowDelimiter=';'])
```
Create a formatter function for CSV format.

**Arguments**

- 'format' is a string array.
- 'delimiter' is the separator between two columns. The default value is ','. 
- 'rowDelimiter' is the separator between two lines. The default value is ';'. 

**Example**
```
def createT(n) {
    return table(take([false, true], n) as bool, take('a'..'z', n) as char, take(short(-5..5), n) as short, take(-5..5, n) as int, take(-5..5, n) as long, take(2001.01.01..2010.01.01, n) as date, take(2001.01M..2010.01M, n) as month, take(time(now()), n) as time, take(minute(now()), n) as minute, take(second(now()), n) as second, take(datetime(now()), n) as datetime, take(now(), n) as timestamp, take(nanotime(now()), n) as nanotime, take(nanotimestamp(now()), n) as nanotimestamp, take(3.1415, n) as float, take(3.1415, n) as double, take(`AAPL`IBM, n) as string, take(`AAPL`IBM, n) as symbol)
}
t = createT(100)
f = mqtt::createCsvFormatter([BOOL,CHAR,SHORT,INT,LONG,DATE,MONTH,TIME,MINUTE,SECOND,DATETIME,TIMESTAMP,NANOTIME,NANOTIMESTAMP,FLOAT,DOUBLE,STRING,SYMBOL])
f(t)
```
### 5.2 createCsvParser

**Syntax**
```
mqtt::createCsvParser(schema, [delimiter=','], [rowDelimiter=';'])
```
Create a parser function for CSV format.

**Arguments**

- is an array of column data types.
- 'delimiter'is the separator between two columns. The default value is ','. 
- 'rowDelimiter' is the separator between two lines. The default value is ';'. 

**Example**
```
def createT(n) {
    return table(take([false, true], n) as bool, take('a'..'z', n) as char, take(short(-5..5), n) as short, take(-5..5, n) as int, take(-5..5, n) as long, take(2001.01.01..2010.01.01, n) as date, take(2001.01M..2010.01M, n) as month, take(time(now()), n) as time, take(minute(now()), n) as minute, take(second(now()), n) as second, take(datetime(now()), n) as datetime, take(now(), n) as timestamp, take(nanotime(now()), n) as nanotime, take(nanotimestamp(now()), n) as nanotimestamp, take(3.1415, n) as float, take(3.1415, n) as double, take(`AAPL`IBM, n) as string, take(`AAPL`IBM, n) as symbol)
}
t = createT(100)
f = mqtt::createCsvFormatter([BOOL,CHAR,SHORT,INT,LONG,DATE,MONTH,TIME,MINUTE,SECOND,DATETIME,TIMESTAMP,NANOTIME,NANOTIMESTAMP,FLOAT,DOUBLE,STRING,SYMBOL])
s=f(t)
p = mqtt::createCsvParser([BOOL,CHAR,SHORT,INT,LONG,DATE,MONTH,TIME,MINUTE,SECOND,DATETIME,TIMESTAMP,NANOTIME,NANOTIMESTAMP,FLOAT,DOUBLE,STRING,SYMBOL])
p(s)
```
### 5.3 createCsvFormatter

**Syntax**
```
mqtt::createCsvFormatter()
```
Create a formatter function for JSON format. 

**Arguments**
    None.

**Example**
```
def createT(n) {
    return table(take([false, true], n) as bool, take('a'..'z', n) as char, take(short(-5..5), n) as short, take(-5..5, n) as int, take(-5..5, n) as long, take(2001.01.01..2010.01.01, n) as date, take(2001.01M..2010.01M, n) as month, take(time(now()), n) as time, take(minute(now()), n) as minute, take(second(now()), n) as second, take(datetime(now()), n) as datetime, take(now(), n) as timestamp, take(nanotime(now()), n) as nanotime, take(nanotimestamp(now()), n) as nanotimestamp, take(3.1415, n) as float, take(3.1415, n) as double, take(`AAPL`IBM, n) as string, take(`AAPL`IBM, n) as symbol)
}
t = createT(100)
f = mqtt::createJsonFormatter()
f(t)
```
### 5.4 createJsonParser

```
mqtt::createJsonParser(schema, colNames)
```
Create a parser function for JSON format.

**Arguments**
- 'schema' is a vector of data types for all columns.
- 'colNames' is a string vector indicating column names.

**Example**
```
def createT(n) {
    return table(take([false, true], n) as bool, take('a'..'z', n) as char, take(short(-5..5), n) as short, take(-5..5, n) as int, take(-5..5, n) as long, take(2001.01.01..2010.01.01, n) as date, take(2001.01M..2010.01M, n) as month, take(time(now()), n) as time, take(minute(now()), n) as minute, take(second(now()), n) as second, take(datetime(now()), n) as datetime, take(now(), n) as timestamp, take(nanotime(now()), n) as nanotime, take(nanotimestamp(now()), n) as nanotimestamp, take(3.1415, n) as float, take(3.1415, n) as double, take(`AAPL`IBM, n) as string, take(`AAPL`IBM, n) as symbol)
}
t = createT(100)
f = mqtt::createJsonFormatter()
p = createJsonParser([BOOL,CHAR,SHORT,INT,LONG,DATE,MONTH,TIME,MINUTE,SECOND,DATETIME,TIMESTAMP,NANOTIME,NANOTIMESTAMP,FLOAT,DOUBLE,STRING,SYMBOL],
`bool`char`short`int`long`date`month`time`minute`second`datetime`timestamp`nanotime`nanotimestamp`float`double`string`symbol)
s=f(t)
x=p(s)

```

### An example
```
loadPlugin("/home/qianxj/MQTTClient/PluginMQTTClient.txt"); 
use mqtt; 

loadPlugin("/home/qianxj/MQTTClient/PluginMQTTClient.txt"); 
use mqtt; 

//***************************publish a table****************************************//
MyFormat = take("", 5)
MyFormat[2] = "0.000"
f = createCsvFormatter(MyFormat, ',', ';')

//create a record for every device
def writeData(hardwareVector){
	hardwareNumber = size(hardwareVector)
	return table(take(hardwareVector,hardwareNumber) as hardwareId ,take(now(),
		hardwareNumber) as ts,rand(20..41,hardwareNumber) as temperature,
		rand(50,hardwareNumber) as humidity,rand(500..1000,
		hardwareNumber) as voltage)
}
def publishTableData(server,topic,iterations,hardwareVector,interval,f){
    conn=connect(server,1883,0,f,100)
    for(i in 0:iterations){
	   t=writeData(hardwareVector)
	   publish(conn,topic,t)
	   sleep(interval)
    }
    close(conn)
         
}
host="192.168.1.201"
submitJob("submit_pub1", "submit_p1", publishTableData{host,"sensor/s001",10,100..149,100,f})
publishTableData(host,"sensor/s001",100,0..99,100,f)


//*******************************subscribe : handler is a table************************************************//
p = createCsvParser([INT, TIMESTAMP, DOUBLE, DOUBLE,DOUBLE], ',', ';' )
sensorInfoTable = table( 10000:0,`deviceID`send_time`temperature`humidity`voltage ,[INT, TIMESTAMP, DOUBLE, DOUBLE,DOUBLE])
conn = mqtt::subscribe("192.168.1.201",1883,"sensor/#",p,sensorInfoTable)
```