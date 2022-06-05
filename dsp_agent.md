# DSP_agent

dsp_agent主要与旧的核心框架交互，即发送字符串指令

## DSP_agent代码结构
代码在实验室的服务器上，密码都是:123456
```bash
#内网登录 密码123456
ssh gpf@192.168.1.147
cd legacy_agent
#外网登录
ssh gpf@121.36.149.142 -P 6200
cd legacy_agent
```
代码在src目录，proto/legacy_proto.proto记录了录了所用用到的服务接口，service.h和service.cc是主要服务的实现，main.cc是程序入口。client目录下的client.cc是一个测试的客户端。

首先需要进入legacy_dev容器，在容器里进行编译,我在容器和本地做了文件映射，
legacy_agent/这个文件夹下的映射到了容器里的/code
```bash
docker start legacy_dev #若容器关闭才需要运行该指令
docker exec -it legacy_dev bash
cd /code/
make
```
若更改了.proto文件需要运行 bash build.sh指令，该脚本会重新生成代码，然后make。

## 代码工作流程
所有的服务都是向核心框架发送指令，因此在服务启动时会通过tcp创建一个连接，见main.cc文件。
```c++
sock = socket(AF_INET, SOCK_STREAM, 0);
//向服务器（特定的IP和端口）发起请求
struct sockaddr_in serv_addr;
memset(&serv_addr, 0, sizeof(serv_addr));  //每个字节都用0填充
serv_addr.sin_family = AF_INET;  //使用IPv4地址
serv_addr.sin_addr.s_addr = inet_addr(ssdp_server_ip.c_str());  //具体的IP地址
serv_addr.sin_port = htons(ssdp_server_port);  //端口

if((connect(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr))) < 0){
    Log(ERROR,"SSDP Servere connect error!");
}
```
之后向这个sock写入数据即可，该变量是全局的。
发送指令后会接收一个回应，如果是load指令返回值会带有一个@的后缀表示重构时间，这时会取出重构时间
见函数sendIns

## 在58部署
1. 启动核心框架，ip是10.119.84.190

2. 启动dsp_agent，用来访问核心框架,在191机器
```bash
ssh ubuntu@192.168.3.193
ssh ubuntu@192.168.1.191
docker start dsp_agent2
docker exec -it dsp_agent2 sh
```
进入容器后/home目录
```bash
./main 192.168.1.191 10
```

## DSP_agent的配置
在58所机器上dsp_agent的配置文件为容器里/home/config.json,其含义如下：
```json
{
	"REGISTRY_CENTER":"192.168.1.10:8500",
    "agent_ip": "192.168.1.191",
    "agent_port": 5103,
    "agent_health_port":5203,
    "service_name": "DSP_agent",
    "weight": 1,
    "ssdp_server_ip": "10.119.84.190", //核心框架ip
    "ssdp_server_port": 8080, //核心框架端口
    "is_regist_service": 1
}

```
这部分和实验室的开发机上的config.json不一样，以58所的为准，当删除一个dsp_agent容器后再次创建时需要修改这个配置文件。如果只是重启容器则不用管。

## 注意事项
1. dsp_agent代码修改后需要打包成容器放到58的板子上，由于编译使用的容器较大，需要把编译好的可执行程序复制到运行时容器上，然后再打包成镜像。步骤如下：
```bash
#在legacy_dev容器中生成两个可执行文件，main和dsp_client
#运行dsp_agent_release容器
docker exec -it dsp_agent_release sh
#将/code目录里的两个可执行文件main和dsp_client拷贝到/home目录
#用docker commit 指令创建一个新的镜像
 docker commit -a hanxi739    dsp_agent_release hanxi739/alpine_dsp_agent:<version>
```
