# SAR元应用相关


## SAR代码结构
1. 代码位置与文件介绍
代码在实验室的服务器上，密码都是:123456
```bash
#内网登录 密码123456
ssh gpf@192.168.1.147
cd app_v3/sar_app
#外网登录
ssh gpf@121.36.149.142 -P 6200
cd app_v3/sar_app
```
其中sar_application.proto文件记录了所用用到的服务接口，server.cc是服务器端代码，client.cc是用来测试的客户端。我们主要对外提供的接口就两个：
```proto
service SAR {
    rpc SAR_Open_Interface(SAR_Open_Request) returns (SAR_Open_Response);
    rpc SAR_Close_Interface(SAR_Close_Request) returns (SAR_Close_Response);
};
```
主要修改这两个接口生成函数的代码:

2. Open_Sar主要的工作（按顺序）
    1. 调用Load Sar接口
    ```c++
    client.LoadSarApp(&req, &resp, &target_Location_synctx);
    ```
    2. sleep一定的时间
    3. 调用Start Sar接口
    ```c++
    client.StartSarApp(&req, &resp, &target_Location_synctx);
    ```
    4. 调用noise_reduction服务
    ```c++
    sar_arm_client.noise_reduction(&noireq, &noiresp, &target_Location_synctx);
    ```
    5. 调用feature_etraction服务
    ```c++
    sar_arm_client.feature_etraction(&feature_req, &feature_resp, &target_Location_synctx);
    ```


3. Close_Sar主要的工作（按顺序）
    1. 调用Stop Sar接口
    ```c++
    client.StopSarApp(&req, &resp, &target_Location_synctx);
    ```
测试时load和start和stop的步骤所需要的的服务需要我们自己启动，即DSP_agent,另外两个服务需要西电同学启动

4. 代码编译流程
首先需要进入sar_app容器，在容器里进行编译,我在容器和本地做了文件映射，
sar_app/这个文件夹下的映射到了容器里的/code
```bash
docker start sar_app_dev #若容器关闭才需要运行该指令
docker exec -it sar_app bash
cd /code/
make
```
若更改了.proto文件需要运行 bash build.sh指令，该脚本会重新生成代码，然后make。

## SAR(雷达在58所的部署测试)

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
让西电的张亚聪同学启动noise_reduction和feature_extration两个服务

2. 首先登录到新主控，然后登录到gpp4
```bash
ssh ubuntu@192.168.3.193
ssh ubuntu@192.168.1.197
```
3. 启动容器sar_app，然后进入
```bash
docker start sar_app
docker exec -it sar_app bash
```

4. 进入sar_app容器的/code，启动server
```bash
./start_server
```

5. 进入sar_app容器的/code，启动client
```bash
./start_client
```

6. 在197机器的/home/ubuntu/sar_app目录观看日志，
```bash
tail -f log.txt
```

## SAR的配置
在58所机器上sar的配置文件为容器里/code/config.json,其含义如下：
```json
{
    "REGISTRY_CENTER":"192.168.1.10:8500", //服务注册中心地址
	"dsp_agent_name":"DSP_agent",  //dsp agent具体的名字
    "sar_arm_name":"SAR_metaservice", //当前服务使用的名字
    "sar_name":"SAR_metaApp", //西电两个应用服务所用的名字
    "sar_port":5100, //当前服务端口
    "check_health_port":5200, //检查服务所用的端口
    "is_regist_service": 1 //是否注册服务
}
```
这部分和实验室的开发机上的config.json不一样，以58所的为准，当删除一个sar容器后再次创建时需要修改这个配置文件。如果只是重启容器则不用管。

## 注意事项
1. 改这部分的代码最好还是在58所的机器上该，实验室服务器上的只是用来备份与学习
2. 关于docker容器，该代码需要在 hanxi739/alpine_srpc_developing:3.0这个镜像的环境中运行，当前的sar_app_dev就是该镜像的容器
3. 关于DSP_agent，文档[dsp_agent](dsp_agent.md), 我会交给hz。
