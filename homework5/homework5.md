# ДЗ 5
### 1

Установил pgbouncer
```sql
sudo apt update
sudo apt install pgbouncer
```

### 2

Установил настройки pgbouncer
```sql
sudo nano pgbouncer.ini 
sudo nano userlist.txt 
sudo systemctl stop pgbouncer
sudo systemctl start pgbouncer
```


### 3

Инициализировал pgbench
```sql
pgbench -i -h 127.0.0.1 -p 6432 -U postgres -d thai
```


### 4

Запустил pgbench для pgBouncer в режиме transaction
```sql
pgbench -h 127.0.0.1 -p 6432 -U postgres -d thai -c 20 -j 4 -T 60

transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 20
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 47182
number of failed transactions: 0 (0.000%)
latency average = 25.439 ms
initial connection time = 4.260 ms
tps = 786.185687 (without initial connection time)
```

### 5

Запустил pgbench для pgBouncer в режиме session
```sql
pgbench -h 127.0.0.1 -p 6432 -U postgres -d thai -c 20 -j 4 -T 60

transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 20
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 49963
number of failed transactions: 0 (0.000%)
latency average = 24.024 ms
initial connection time = 3.919 ms
tps = 832.516682 (without initial connection time)

```

### 6

Запустил pgbench для pgBouncer в режиме statement
```sql
pgbench -h 127.0.0.1 -p 6432 -U postgres -d thai -c 20 -j 4 -T 60

pgbench: client 13 receiving
pgbench: error: client 13 script 0 aborted in command 4 query 0: FATAL:  transaction blocks not allowed in statement pooling mode
server closed the connection unexpectedly
	This probably means the server terminated abnormally
	before or while processing the request.
pgbench: client 14 receiving
pgbench: error: client 14 script 0 aborted in command 4 query 0: FATAL:  transaction blocks not allowed in statement pooling mode
server closed the connection unexpectedly
	This probably means the server terminated abnormally
	before or while processing the request.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 20
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 0
number of failed transactions: 0 (NaN%)
pgbench: error: Run was aborted; the above results are incomplete.
                                 
                                 
pgbench -h 127.0.0.1 -p 6432 -U postgres -d thai -c 20 -j 4 -T 60 -S   
                                 
transaction type: <builtin: select only>
scaling factor: 1
query mode: simple
number of clients: 20
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 434834
number of failed transactions: 0 (0.000%)
latency average = 2.760 ms
initial connection time = 4.601 ms
tps = 7247.474916 (without initial connection time)
```