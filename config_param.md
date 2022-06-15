# 参数说明
通信(com)，扩频(nav)，电子干扰(jam)三个应用都有自己的config指令，调用config指令时会下发不同的字符串参数

## 1、通信模块
configRequest消息在proto文件的定义:
```proto
message Comm_Config_Request{
    int32 type = 1; // "ldpc" or "turbo"
};
```
|  type取值   | config 字符串  | 意义 |
|  ----  | ----  | ----  |
| 0  | Turbo@444c0f9a | Turbo编码 （Start默认模式）  |
| 1  | LDPC@444c0f9a | LDPC编码 |

## 2、导航（扩频）模块
configRequest消息在proto文件的定义:
```proto
message Nav_Config_Request{
    int32 type = 1; 
};
```
|  type取值   | config 字符串  | 意义 |
|  ----  | ----  | ----  |
| 0  | lkp@444c0f9a | 低速扩频  （Start默认模式） |
| 1  | hkp@444c0f9a | 高速扩频 |

## 3、电子干扰模块
configRequest消息在proto文件的定义:
```proto
message Jam_Config_Request{
    int32 useful = 1; //是否有用信号， 0：无用，1：有用
    int32 noise = 2; //0: 无， 1:梳状谱干扰，2：高斯白噪声
    int32 isr = 3; //irs取值
};
```
现阶段可取值如下：
|  useful   | noise  | isr |config 字符串  | 意义 |
|  ----  | ----  | ----  | ---|---|
| 1  | 1 | 1 |usesig_shu_1@69a294d8 | 有用信号 + 梳状谱干扰，ISR=1dB  （default） |
| 1  | 2 | 10 |usesig_gauss_10@69a294d8| 有用信号 + 高斯白噪声, ISR=10dB |



# 参数与FPGA对应
config指令的目的是向核心框架发送不同的字符串配置，核心框架得到配置后会根据config字符串下发送不同的值（int类型）发送到FPGA的elf文件。以电子干扰为例，若发送的字符串是usesig_gauss_10@69a294d8，核心框架jam.xml配置文件部分如下:
```xml
<component>
			<objId>usesig_gauss_10@69a294d8</objId>
			<componenId>0x1a</componenId>
			<resourceInfo>
				<info>
					<category>fpga</category>
					<codeLocation>/opt/upgrade/download_kp.bit</codeLocation>
					<resourceUsed>12%</resourceUsed>
					<name>fpga1</name>
				</info>
			</resourceInfo>
			<parameters>
				<parameter>
					<name>framelength</name>
					<address>0x06</address>
					<value>0x00</value>
				</parameter>
			</parameters>
		</component>
```
componenId为0x1a，FPGA的elf文件收到该数值后会写入信号，核心代码如下:
```
    else if(value == 0xa){
        Xil_OUT(address, 8800);
        Xil_OUT(address, 8801);
        Xil_OUT(address, 8800);
    }
```

# 西工大提供的最新配置文件如下:
[西工大提供的信号](xgd_file.txt)