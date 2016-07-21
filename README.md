# How to config rabbitmq server cluster for SoftnixLogger [3 nodes]

## Do after install SLGInstallers

## Edit /etc/hosts
```
vi /etc/hosts
192.168.10.157  rabbitmq-1
192.168.10.159  rabbitmq-2
192.168.10.161  rabbitmq-3
```

### Restart rabbitmq-server [all nodes]
```
service rabbitmq-server restart
```

### Enable rabbitmq-server service
```
chkconfig rabbitmq-server on
```

#### Verify rabbitmq-server is running [all nodes]
```
rabbitmqctl status
Result
Status of node 'rabbit@rabbitmq-1' ...
[{pid,2163},
 {running_applications,[{rabbit,"RabbitMQ","3.6.0"},
                        {ranch,"Socket acceptor pool for TCP protocols.",
                               "1.2.1"},
                        {rabbit_common,[],"3.6.0"},
                        {xmerl,"XML parser","1.3.8"},
                        {mnesia,"MNESIA  CXC 138 12","4.13.1"},
                        {os_mon,"CPO  CXC 138 46","2.4"},
                        {sasl,"SASL  CXC 138 11","2.6"},
                        {stdlib,"ERTS  CXC 138 10","2.6"},
                        {kernel,"ERTS  CXC 138 10","4.1"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang/OTP 18 [erts-7.1] [source] [64-bit] [smp:8:8] [async-threads:64] [hipe] [kernel-poll:true]\n"},
 {memory,[{total,46149608},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,0},
          {queue_procs,2808},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,18729656},
          {mnesia,60536},
          {mgmt_db,0},
          {msg_index,47200},
          {other_ets,871984},
          {binary,24216},
          {code,17244116},
          {atom,662409},
          {other_system,8506683}]},
 {alarms,[]},
 {listeners,[{clustering,25672,"::"},{amqp,5672,"::"}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,6713846988},
 {disk_free_limit,50000000},
 {disk_free,583758884864},
 {file_descriptors,[{total_limit,924},
                    {total_used,2},
                    {sockets_limit,829},
                    {sockets_used,0}]},
 {processes,[{limit,1048576},{used,135}]},
 {run_queue,0},
 {uptime,244},
 {kernel,{net_ticktime,60}}]
```

### Create rabbitmq-server cluster
#### Step 1 view /var/lib/rabbitmq/.erlang.cookie
```
[Node 1]
cat /var/lib/rabbitmq/.erlang.cookie
Result = AFYDPNYXGNARCABLNENP
```

#### Step 2 set cookie same node 1 (Make sure the erlang cookies are the same all node.)
```
[Node 2-3]
service rabbitmq-server stop
echo -n "AFYDPNYXGNARCABLNENP" > /var/lib/rabbitmq/.erlang.cookie
service rabbitmq-server start
```

#### Step 3 join cluster to node 1
```
[Node 2-3]
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@rabbitmq-1
rabbitmqctl start_app
```

#### Verify rabbitmq-server cluster [all nodes]
```
rabbitmqctl cluster_status
Result
Cluster status of node 'rabbit@rabbitmq-1' ...
[{nodes,[{disc,['rabbit@rabbitmq-1','rabbit@rabbitmq-2',
                'rabbit@rabbitmq-3']}]},
<pre>
 <b>{running_nodes,['rabbit@rabbitmq-3','rabbit@rabbitmq-2','rabbit@rabbitmq-1']},</b>
</pre>
 {cluster_name,<<"rabbit@rabbitmq-1">>},

 {partitions,[]}]
```