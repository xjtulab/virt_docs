# 通信元应用相关


## 通信代码结构
1. 代码位置与文件介绍
代码在实验室的服务器上，密码都是:123456
```bash
#内网登录 密码123456
ssh gpf@192.168.1.147
cd app_v3/com_app
#外网登录
ssh gpf@121.36.149.142 -P 6200
cd app_v3/com_app
```
其中com_application.proto文件记录了所用用到的服务接口，server.cc是服务器端代码，client.cc是用来测试的客户端。我们主要对外提供的接口就两个：
```proto
service CommApplication {
    rpc Comm_Open_Interface(Comm_Open_Request) returns (Comm_Open_Response);
    rpc Comm_Close_Interface(Comm_Close_Request) returns (Comm_Close_Response);
};
```
主要修改这两个接口生成函数的代码:

2. Open_Comm主要的工作（按顺序）
    1. 调用Load Comm接口
    ```c++
    client.LoadCommunication(&req, &resp, &target_Location_synctx);
    ```
    2. sleep一定的时间
    3. 调用Conif Comm接口，在FPGA端实际上是start
    ```c++
    client.ConfigCommunication(&req, &resp, &target_Location_synctx);
    ```



3. Close_Comm主要的工作（按顺序）
    1. 调用Stop Comm接口
    ```c++
    client.StopCommunication(&req, &resp, &target_Location_synctx);
    ```


4. 代码编译流程
首先需要进入comm_app容器，在容器里进行编译,我在容器和本地做了文件映射，
com_app/这个文件夹下的映射到了容器里的/code
```bash
docker start comm_app_dev #若容器关闭才需要运行该指令
docker exec -it comm_app_dev bash
cd /code/
make
```
若更改了.proto文件需要运行 bash build.sh指令，该脚本会重新生成代码，然后make。

## 通信模块部署(58所的机器)

1. 启动dsp_agent，用来访问核心框架,在191机器
```bash
ssh ubuntu@192.168.3.193
ssh ubuntu@192.168.1.191
docker start dsp_agent
docker exec -it dsp_agent sh
```
进入容器后/home目录
```bash
./main 192.168.1.191 10
```

2. 首先登录到新主控，然后登录到gpp4
```bash
ssh ubuntu@192.168.3.193
ssh ubuntu@192.168.1.197
```
3. 启动容器com_app，然后进入
```bash
docker start com_app
docker exec -it com_app bash
```

4. 进入com_app容器的/code，启动server
```bash
./com_server 192.168.1.197 10
```

5. 进入com_app容器的/code，启动client
```bash
./com_client 192.168.1.197 5106
```

6. 在197机器的/home/ubuntu/com_app目录观看日志，
```bash
tail -f log.txt
```

## 注意事项
1. 改这部分的代码最好还是在58所的机器上该，实验室服务器上的只是用来备份与学习
2. 通信模块启动的参数由下表定义，在实现中只需要给核心框架发送一个32位的整数值作为参数即可，具体的移位操作见代码。有关通信功能的问题@喻永松同学
-----
| 名称              | 位宽 |  功能 |
| -----------       | ----------- | --- |
| rd_start          | [0:0]     |    0:不启动 1：启动      |
| BitRate           | [2:1]      | 0：3.125Mbps(默认) <br> 1：6.25Mbps<br>2：12.5Mbps<br>3：25Mbps|
| ModulationOrder   | [6:3]     | 0：2阶<br>1：4阶(默认)<br>2：8阶<br>3：16阶<br>4：64阶 |
| SpreadSpectrumRate  | [7:7]        | 0：1.023Mbps(默认)<br>1：10.23Mbps |
 | CodeType          | [8:8]        |  0：编码1<br>1：编码2|
| ModType           | [9:9]        | 0：调制1（Sar成像）<br>1：调制2（导航增强） |

