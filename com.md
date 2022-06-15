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
    rpc Comm_Config_Interface(Comm_Config_Request) returns (Comm_Config_Response);
    rpc Comm_Get_Reconstruct_Time(Comm_ReconstructTime_Request) returns (Comm_ReconstructTime_Response);
};
```
主要修改这两个接口生成函数的代码:

2. Open_Comm主要的工作（按顺序）
    1. 调用Load Comm接口
    ```c++
    client.LoadCommunication(&req, &resp, &target_Location_synctx);
    ```
    2. sleep一定的时间
    3. 调用Start Comm接口，默认是Turbo编码
    ```c++
    client.StartCommunication(&req, &resp, &target_Location_synctx);
    ```


3. Close_Comm主要的工作（按顺序）
    1. 调用Stop Comm接口
    ```c++
    client.StopCommunication(&req, &resp, &target_Location_synctx);
    ```

4. Config_Comm主要的工作（按顺序）
    1. 调用Config Comm接口，根据传入type类型设置不同的objid
    ```c++
    string obj_id = "Turbo@444c0f9a";
		if(request->type() == 1)
			obj_id = "LDPC@234bfc8c";
		
		req.set_objid(obj_id);
    client.ConfigCommunication(&req, &resp, &target_Location_synctx);
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

1. 首先登录到新主控，然后登录到gpp4
```bash
ssh ubuntu@10.119.84.100
ssh ubuntu@192.168.1.194
```
2. 启动容器com_app，然后进入
```bash
docker start com_app
docker exec -it com_app bash
```

3. 进入com_app容器的/code，启动server
```bash
./deploy.sh
```

4. 进入com_app容器的/code，启动client
```bash
./com_client 192.168.1.194 5106
```

5. 在194机器的/home/ubuntu/com_app目录观看日志，
```bash
tail -f log.txt
```



