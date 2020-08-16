# Week1

## 部署pd

* 安装go，并配置环境变量
```
wget https://dl.google.com/go/go1.15.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.15.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```

* 编译
```
make
```

* 启动
```
cd bin
nohup ./pd-server --log-file=pd.log --name=pd --data-dir=pd > nohup.log 2>&1 &
```

* 日志
```
[2020/08/16 15:29:34.124 +08:00] [INFO] [server.go:1150] ["PD cluster leader is ready to serve"] [leader-name=pd]
```

* dashboard
```
http://127.0.0.1:2379/dashboard
```

* 配置IDEA环境
```
安装go plugin
配置GOROOT和GOPATH
```

## 部署tikv

* 安装rust，并配置环境变量
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
export PATH=$HOME/.cargo/bin:$PATH
```

* 编译
```
rustup component add clippy
make build
```

* 启动
```
cd target/debug
nohup ./tikv-server --log-file tikv1.log --addr 127.0.0.1:20160 --data-dir=tikv1 > nohup1.log 2>&1 &
nohup ./tikv-server --log-file tikv2.log --addr 127.0.0.1:20260 --data-dir=tikv2 > nohup2.log 2>&1 &
nohup ./tikv-server --log-file tikv3.log --addr 127.0.0.1:20360 --data-dir=tikv3 > nohup3.log 2>&1 &
```

* 日志
```
tikv1:
[2020/08/16 15:34:05.721 +08:00] [INFO] [server.rs:219] ["listening on addr"] [addr=127.0.0.1:20160]
[2020/08/16 15:34:05.761 +08:00] [INFO] [server.rs:244] ["TiKV is ready to serve"]

tikv2:
[2020/08/16 15:34:05.716 +08:00] [INFO] [server.rs:219] ["listening on addr"] [addr=127.0.0.1:20260]
[2020/08/16 15:34:05.755 +08:00] [INFO] [server.rs:244] ["TiKV is ready to serve"]

tikv3:
[2020/08/16 15:34:05.737 +08:00] [INFO] [server.rs:219] ["listening on addr"] [addr=127.0.0.1:20360]
[2020/08/16 15:34:05.771 +08:00] [INFO] [server.rs:244] ["TiKV is ready to serve"]
[2020/08/16 15:34:15.831 +08:00] [INFO] [raft.rs:985] ["became leader at term 7"] [term=7] [raft_id=9] [region_id=3]
```

* 验证
```
./pd-ctl store

{
  "count": 3,
  "stores": [
    {
      "store": {
        "id": 1,
        "address": "127.0.0.1:20160",
        "version": "4.1.0-alpha",
        "status_address": "127.0.0.1:20180",
        "git_hash": "df2fab45b19b5d067ff55ff5074819c50eef3ea2",
        "start_timestamp": 1597563245,
        "deploy_path": "/home/meiyi/workspace/tikv/target/debug",
        "last_heartbeat": 1597563705755607148,
        "state_name": "Up"
      },
      "status": {
        "capacity": "409.6GiB",
        "available": "301.4GiB",
        "used_size": "31.5MiB",
        "leader_count": 0,
        "leader_weight": 1,
        "leader_score": 0,
        "leader_size": 0,
        "region_count": 1,
        "region_weight": 1,
        "region_score": 1,
        "region_size": 1,
        "start_ts": "2020-08-16T15:34:05+08:00",
        "last_heartbeat_ts": "2020-08-16T15:41:45.755607148+08:00",
        "uptime": "7m40.755607148s"
      }
    }
    ...
}
```

* 配置IDEA环境
```
安装rust plugin
```

## 部署tidb

* 编译
```
make
```

* 启动
```
cd bin
nohup ./tidb-server --log-file=tidb.log --path=tidb > nohup.log 2>&1 &
```

* 日志
```
[2020/08/16 15:47:20.949 +08:00] [INFO] [server.go:235] ["server is running MySQL protocol"] [addr=0.0.0.0:4000]
```

## 在事务启动时，打印日志

在[kv/kv.go](https://github.com/mymeiyi/tidb/blob/07ae6078e887f42ff74a2787a9f9db3895f639d4/kv/kv.go#L427) 的Storage#Begin或Storage#BeginWithStartTS:
```
type Storage interface {
	// Begin transaction
	Begin() (Transaction, error)
	// BeginWithStartTS begins transaction with startTS.
	BeginWithStartTS(startTS uint64) (Transaction, error)
    ...
}
```

这两个方法最终都调用[txn/txn.go](https://github.com/mymeiyi/tidb/blob/07ae6078e887f42ff74a2787a9f9db3895f639d4/store/tikv/txn.go#L96) 的newTikvTxnWithStartTS方法创建事务，因此，在这里打印"hello transaction":
```
// newTikvTxnWithStartTS creates a txn with startTS.
func newTikvTxnWithStartTS(store *tikvStore, startTS uint64, replicaReadSeed uint32) (*tikvTxn, error) {
	logutil.BgLogger().Info("hello transaction")
    ...
}
```
