# 使用快照

本快照需要eosforce node v1.8以上版本


## 1、下载最新版docker image
```
docker pull eosforce/node:v1.8.3
```

## 2、下载最近的快照文件到服务器

wget -c https://raw.githubusercontent.com/eosforce/node-snapshots/master/xxxx.tar.gz

注意下载的url，不是 https://github.com/ 开头的，将实际的文件名，替换上面的 xxxx.tar.gz

## 3、解压下载的文件，并移动复制到节点数据目录(以node_data为例)
```
tar -zxvf 00ec05ba1d16aafae795ce3bbfff4313a5130e8aafcb976bdacd54764d3b1198.tar.gz
mv snapshot-00ec05ba1d16aafae795ce3bbfff4313a5130e8aafcb976bdacd54764d3b1198.bin  /node_data/snapshot.bin 
```

## 4、修改config.ini文件，加载producer_api_plugin
```config.ini
plugin = eosio::producer_plugin
plugin = eosio::producer_api_plugin
```

## 5、准备好节点的数据目录
目录格式如下  
├── node_data     
│       ├── snapshot.bin   
│       ├── config   
│       │      ├── activeacc.json      
│       │      ├── config.ini   
│       │      ├── eosio.lock.abi   
│       │      ├── ...   
│       │      └── genesis.json   
│       ├── data   
│       │   (此目录一定要为空)

## 6、先启动一个预备容器来处理快照数据

```
docker run -d --name eosforce-snapshot \
-v /node_data:/eosforce \
eosforce/node:v1.8.3 /opt/eosio/bin/nodeos \
--config-dir=/eosforce/config \
--data-dir=/eosforce/data \
--snapshot=/eosforce/snapshot.bin
```

相关参数说明
| 名称   | 描述                                    | 补充说明                                                                                                            |
|:-------|:----------------------------------------|:--------------------------------------------------------------------------------------------------------------------|
| -d     | 指定容器运行于前台还是后台，默认为false | 将node节点在后台运行                                                                                                |
| --name | 给容器命名                              | --name eosforce-snapshot 是给容器起名为eosforce-snapshot                                                            |
| -v     | 给容器挂载存储卷，挂载到容器的某个目录  | -v /node_data:/eosforce 是将宿主机的/node_data目录挂载到docker里面的/eosforce，这个目录就是宿主机和docker共享的 |
| -p     | 指定容器暴露的端口                      | -p 9876:9876 将docker的39876端口挂载到宿主机的9876端口，这样可以通过宿主机的9876端口访问到docker的9876端口     |
|/opt/eosio/bin/nodeos --config-dir=/eosforce/config --data-dir=/eosforce/data --snapshot=/eosforce/snapshot.bin| 启动node节点的命令，这里面指定了--config-dir，--data-dir，--snapshot 三个参数，地址需要时docker里面目录地址。||



## 7、查看容器日志，等到同步完成时。停止预备容器
```
docker logs -f --tail 100 eosforce-snapshot

(ctrl+c 退出日志状态)

docker stop eosforce-snapshot

```

## 8、先启动一个正式容器来运行节点

```
docker run -d --name eosforce-node \
-v /node_data:/eosforce \
-p 9876:9876 -p 8888:8888 \
eosforce/node:v1.8.3 nodeosd.sh
```

## 9、查看容器日志，确认已块出后。删除预备容器
```
docker logs -f --tail 100 eosforce-node

(ctrl+c 退出日志状态)

docker rm eosforce-snapshot

```

## 10、生成快照文件
```
curl -XPOST http://127.0.0.1:8005/v1/producer/create_snapshot
```

## 参考文档
EOS 主网节点搭建,快照snapshots同步数据 https://www.jianshu.com/p/27c81cf4a107  

EOS的快照(snapshot)功能 https://hive.blog/hive-105017/@oflyhigh/eos-snapshot





