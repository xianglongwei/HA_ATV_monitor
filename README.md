# HA_ATV_monitor
# APPLE TV 联动电视开机关机
电视没有ECC功能，无法用一个遥控器进行开关，可使用此方法

# 使用docker 项目：
## lscr.io/linuxserver/homeassistant
```
docker run -d \
  --name homeassistant \
  --network host \  # 使用 Host 模式
  -e TZ=Asia/Shanghai \  # 设置时区
  -v ~/homeassistant/config:/config \
  --restart unless-stopped \
  lscr.io/linuxserver/homeassistant:latest

```
HA初始配置请自行搜索

## eclipse-mosquitto:2
1|拉取 eclipse-mosquitto:2 镜像
```
docker pull eclipse-mosquitto:2

```
2. 创建 Mosquitto 配置文件
```
mkdir -p ~/mosquitto/config ~/mosquitto/data ~/mosquitto/log
nano ~/mosquitto/config/mosquitto.conf
```
然后写入以下内容：mqtt config文件
```
listener 1883 
allow_anonymous true 
persistence true  
persistence_location /mosquitto/data/

log_dest stdout 
log_type all    
log_timestamp true
```
运行 Mosquitto 容器
```
docker run -d \
  --name mosquitto \
  --network host \  
  -v ~/mosquitto/config/mosquitto.conf:/mosquitto/config/mosquitto.conf \
  -v ~/mosquitto/data:/mosquitto/data \
  -v ~/mosquitto/log:/mosquitto/log \
  eclipse-mosquitto:2

```






## sebbo2002/pyatv-mqtt-bridge
创建配置文件/mnt/cached/config/config.json
```
{
  "broker": "mqtt://192.168.1.1",
  "devices": [
    {
      "name": "Living Room Apple TV",
      "topic": "home/livingroom/appletv",
      "host": "192.168.1.2",
      "id": "8xxxxxF4C0-xxxxxxxxxx29472CA285",
      "airplayCredentials": "xxxxxxxxxxxxxxxxxxxxx5"
    }
  ]
}

```
运行
```
docker run -d --restart=always --name=pyatv-mqtt-bridge \
  --net=host \
  -v /mnt/cached/config/config.json:/app/config.json:ro \
  sebbo2002/pyatv-mqtt-bridge \
  pyatv-mqtt-bridge --debug /app/config.json

```


进入pyatv-mqtt-bridge控制台:
执行以下命令来查看特定 Apple TV 设备的详细信息：
```

atvremote --id <AppleTV-ID> info
```
用 atvremote scan 得到的 ID 来替换 <AppleTV-ID>，例如：
```

atvremote --id xxxxxxxxxxxxxxxxxxxxx info
```


```
获取 AirPlay Credentials（配对步骤）
使用 atvremote 命令进行配对（确保你选择的设备 ID 与配置文件中的一致）：

atvremote --id xxx-xxx-xxx-xx-xxxx --protocol airplay pair
按照提示完成配对，配对成功后会输出类似如下格式的凭据字符串：
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA:BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB:CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC:DDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDD
将该字符串复制到配置文件中的 "airplayCredentials" 字段，替换掉空字符串。
```
修改获取的信息：然后重启
```
{
  "broker": "mqtt://192.168.1.1:xxxx", 如果没有端口号就删除 ":xxx"
  "devices": [
    {
      "name": "appletv_xxxxx",  #取个名字
      "topic": "home/livingroom/appletv/powerState",
      "host": "192.168.10.107", #atv_IP地址
      "id": "xxxxxxxxxxx", # atv ID
      "airplayCredentials": "xxxxxxxxxxxx"
    }
  ]
}



```


### HA_MQTT配置:
1|HA 配置MQTT：
![image](https://github.com/user-attachments/assets/98a26683-93ac-42f4-8a25-fe4ef9b5df5b)

![image](https://github.com/user-attachments/assets/13edc4cf-def3-4223-8023-134cc1323d02)
如果配置了账号密码请输入上去--其他都是默认

进入homeassistant的配置文件configuration.yaml 在最下面添加实体

```
mqtt:
  binary_sensor: 
      name: "Apple TV 状态"
      state_topic: "home/livingroom/appletv/powerState/powerState"  #这个是监控ATV开机关机的其他请查看项目说明[pyatv-mqtt-bridge](https://github.com/sebbo2002/pyatv-mqtt-bridge)
      payload_on: "on"
      payload_off: "off"
      device_class: "power"
```

## 配置完毕后请查看是否生效
![image](https://github.com/user-attachments/assets/8271ae7d-b7da-4900-b212-e5f2725c3c6e)
## 如果生效就可以配置电视开机关机同步自动化了
![image](https://github.com/user-attachments/assets/1cb73b88-8c19-4269-b718-045c7925a943)



