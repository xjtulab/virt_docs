# 导航与电子干扰
两部分代码与通信代码类似，位置在实验室的服务器上
```bash
#内网登录 密码123456
ssh gpf@192.168.1.147
cd app_v3/nav_app
cd app_v3/jam_app
#外网登录
ssh gpf@121.36.149.142 -P 6200
...
```

58所在192.168.1.194的板子上
```bash
ssh ubuntu@10.119.84.88
ssh ubuntu@192.168.1.194
```
启动容器com_app，然后进入
```bash
docker start com_app
docker exec -it nav_app bash
docker exec -it jam_app bash
```

# 不同点
其逻辑与通信的类似，只不过config接口发送的字符串有变化。