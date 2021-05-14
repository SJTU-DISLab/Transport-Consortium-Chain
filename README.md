# 相关文件说明

1. meta

  元文件夹,需要存放fisco bcos二进制文件、链证书ca.crt、本机构机构证书agency.crt、机构私钥节点证书、群组创世区块文件等.证书的存放格式需要为cert_p2pip_port.crt的格式，如cert_127.0.0.1_30300.crt。之后generator会根据用户在元数据文件夹下放置的相关证书、conf下的配置文件，生成用户下配置的节点配置文件夹。
  
2. tpl/group_genesis.ini 

  该文件涉及对共识算法,存储状态,gas上限的配置. 
  
3. node_deployment.ini 

  通过修改该文件的配置,用户可以使用–build_install_package命令在指定文件夹下生成节点不含私钥的节点配置文件夹.读取节点配置的命令，如生成节点证书和节点配置文件夹等会读取该配置文件。
  
  为便于开发和体验，channel_listen_ip参考配置是 0.0.0.0 ，出于安全考虑，请根据实际业务网络情况，修改为安全的监听地址，如：内网IP或特定的外网IP
  
  RPC/P2P/Channel监听端口必须位于1024-65535范围内，且不能与机器上其他应用监听端口冲突;
# 交通联盟链两服务器部署过程
1. 本次部署服务器1外网地址为101.132.114.148，服务器2外网地址为120.78.95.105.
2. 对服务器1和2都执行以下操作，以下载fisco
  git clone https://github.com/FISCO-BCOS/generator.git && cd ./generator
3. 执行下面操作，以安装fisco
  bash ./scripts/install.sh
4. 检查是否安装成功,若成功，输出 usage: generator xxx
  ./generator -h
5. 将右侧的release发行版二进制文件放入meta文件夹下
6. 到此，fisco下载安装完毕，此时generator文件夹如下：
 <img width="520" alt="图片" src="https://user-images.githubusercontent.com/56347756/115842311-46f91500-a450-11eb-8890-9e724559ea20.png">
 
7. 修改tpl/group.i.genesis文件
   将consensus_type=pbft的pbft修改为tcc
   
此后操作针对一台服务器部署

8. 修改tmp_one_click文件夹下的agencyA和agencyB中的node_deployment.ini,下图是修改后的文件. 
<img width="223" alt="图片" src="https://user-images.githubusercontent.com/56347756/115849235-4617b180-a457-11eb-9c19-057692431966.png">
<img width="197" alt="图片" src="https://user-images.githubusercontent.com/56347756/115849251-49ab3880-a457-11eb-916e-8bd7e1d5e02c.png">

9. 根据node_deployment.ini搭建节点：
bash ./one_click_generator.sh -b ./tmp_one_click

10. 查看tmp_one_click中生成的文件：
ls ./tmp_one_click
<img width="505" alt="图片" src="https://user-images.githubusercontent.com/56347756/115849469-7d865e00-a457-11eb-95d3-d810e73f26f7.png">

11. 启动节点
bash ./tmp_one_click/agencyA/node/start_all.sh
12. 查看节点进程：
ps -ef | grep fisco

13. 查看节点log

tail -f log/*

查看该节点相连的节点数（包括本服务器的其他node）

tail -f log/log* | grep connected

14. 将服务器1中生成的tmp_one_click/agencyB下的node文件夹拷贝到服务器2,拷贝到了和generator文件夹同级的位置.
15. 切入到服务器2的node文件夹，使用./start_all.sh 建立联系
16. 在服务器1和2上，分别进入node/node_xxx.xxx.xxx文件夹下，执行
cat log/* |grep Report 操作，可以看到，两个服务器的hash值相同，说明是同一条链，链的长度等参数也可以看到.

# 性能测试
本测试用于单机测试

1. 节点运行后,将节点生成的sdk文件替换/benchmarks/caliper-benchmarks/networks/fisco-bcos/4nodes1group/下的sdk文件;
2. 修改/benchmarks/caliper-benchmarks/networks/fisco-bcos/4nodes1group/fisco-bcos.json文件:

(1).删除"command"内容:    "command": {

                         }
                          
(2)根据实际ip号,rpcport,channelPort修改对应节点

(3)修改智能合约第一项为:         "smartContracts": [

            {
                "id": "helloworld",
                "address": "0x0000000000000000000000000000000000005001",
                "language": "precompiled",
                "version": "v0"  
            },
            
3.修改benchmarks/caliper-benchmarks/benchmarks/samples/fisco-bcos/helloworld/config.yaml文件: 
  
  将txnumber调整到20000, tps调整为20000, interval调整为0.05

  
