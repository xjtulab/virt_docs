# virt_docs
## Index
G部分文档，主要包含四个元应用和一个DSP_Agent
1. [dsp_agent](dsp_agent.md)
2. [雷达](sar.md)
3. [通信](com.md)
4. [导航与干扰](nav_jam.md)
5. [通信参数说明](config_param.md)
6. [西工大通信接口](xgd_file.txt)



## 代码位置总结
### 实验室环境(树莓派)
|  名字   |  ip地址 | 路径 |
|  ----  | ----  | ----  |
| dsp_agent  | 192.168.1.147 | /home/gpf/legacy_agent |
| 雷达  | 192.168.1.147 | /home/gpf/app_v3/sar_app |
| 通信  | 192.168.1.147 | /home/gpf/app_v3/com_app |
| 导航  | 192.168.1.147 | /home/gpf/app_v3/nav_app |
| 干扰  | 192.168.1.147 | /home/gpf/app_v3/jam_app |


### 58所板子上
|  名字   |  ip地址 | 路径 |
|  ----  | ----  | ----  |
| dsp_agent  | 192.168.1.191 | /home/ubuntu/legacy_agent |
| 雷达  | 192.168.1.194 | /home/ubuntu/sar_app |
| 通信  | 192.168.1.194 | /home/gpf/ubuntu/com_app |
| 导航  | 192.168.1.194 | /home/gpf/ubuntu/nav_app |
| 干扰  | 192.168.1.194 | /home/gpf/ubuntu/jam_app |


### 核心框架代码
[ssdp](https://github.com/xjtulab/ssdp_framework)
注:最新的以58所那台笔记本为准。

## 最终部署 ！！！
1. 启动dsp_agent
```bash
ssh ubuntu@10.119.84.88
ssh ubuntu@192.168.1.191
./start_all.sh
```

2. 启动194机器上的4个服务
```bash
ssh ubuntu@10.119.84.88
ssh ubuntu@192.168.1.194
./start_all.sh
```
