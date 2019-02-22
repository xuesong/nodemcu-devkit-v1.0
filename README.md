NodeMCU DEVKIT V1.0
==============

A development kit for NodeMCU firmware.

It will make NodeMCU more easy. With a micro USB cable, you can connect NodeMCU devkit to your laptop and flash it without any trouble, just like Arduino.

![DEVKIT](https://raw.githubusercontent.com/nodemcu/nodemcu-devkit-v1.0/master/Documents/NodeMCU_DEVKIT_1.0.jpg)

It is an open hardware, with ESP-12-E core [32Mbits(4MBytes) flash version].

## How to flash
- - - - - -
UPDATE
New nodemcu-flasher is released.
Bug fixed. please use latest software and re-flash.
Enjoy.
https://github.com/nodemcu/nodemcu-flasher
- - - - - -
If always have problem, please use latest flash download tool from espressif.
http://bbs.espressif.com/viewtopic.php?f=5&t=433
Please use DIO mode and 32M flash size option, and flash latest firmware to 0x00000.
Before flashing firmware, please hold FLASH button, and press RST button once.
When our firmware download tool released, it will flash firmware automatically and needn't press any button.

## Pin map

![PIN MAP](https://raw.githubusercontent.com/nodemcu/nodemcu-devkit-v1.0/master/Documents/NODEMCU_DEVKIT_V1.0_PINMAP.png)

It is designed by Altium Designer, and fully open–source. Now everyone can make their own NODEMCU.


## Designator & Parameter

|Designator|Parameter|
|-|-|
|C1,C4,C6 |100nF (104) ±10% 16V|
|C2 |100uF(107) ±10% 6.3V Case B 3528|
|C3,C5,C8 |10uF(106) ±20% 10V |
|C7|10uF(106) ±10% 25V|
|D1|SOD-123 40V,1A,VF=0.45V@1A|
|LED1|1.6x0.8x0.6 Iv= 35~65mcd @IF=5mA ESD:1000V |
|R1,R2,R3,R4,R5,R7,R8|12KΩ (1202) ±1% 0.0625W [50V TYP] [100V MAX] T.C.R ±100|
|R6,R9,R11,R12|470Ω (4700) ±1% 0.0625W [50V TYP] [100V MAX] T.C.R ±100|
|R10(Do not install)|0Ω (0R00) ±1% 0.0625W [50V TYP] [100V MAX] T.C.R ±100|
|R13|220kΩ (2203) ±1% 0.0625W [50V TYP] [100V MAX] T.C.R ±100|
|R14|100kΩ (1003) ±1% 0.0625W [50V TYP] [100V MAX] T.C.R ±100|

https://nodemcu.readthedocs.io/en/master/

https://nodemcu-build.com/

https://www.espressif.com/zh-hans/support/download/documents

http://tinylab.org/nodemcu-kickstart/


如果不想搭建编译环境，也可以在线编译。

https://travis-ci.org/ 如果需要修改代码，可以用该方案。只需要把修改提交到自己的 github 仓库，并通过 github 配置好 .travis.yml 即可。

http://nodemcu-build.com 该方案适合直接采用 NodeMCU 源，并且不做修改的情况，它提供了 Web 界面可以简单方便地选择所需模块。




8.1 启动后不断闪烁 LED
上面其实已经演示了 LED 的基本操作，这里再介绍一个 timer module 的 API：tmr.alarm()：

tmr.alarm(id, interval, repeat, function do())

id: 0~6, alarmer id. Interval: alarm time, unit: millisecond
repeat: 0 - one time alarm, 1 - repeat
function do(): callback function for alarm timed out

咱们基于它实现一个 blink.lua:

print('Blink Demo')
lighton=0
led=0
gpio.write(led, gpio.HIGH)
tmr.alarm(0,1000,1,function()
if lighton==0 then
    lighton=1
    gpio.mode(led, gpio.OUTPUT)
    gpio.write(led, gpio.LOW)
else
    lighton=0
    gpio.write(led, gpio.HIGH)
end
end)
gpio.mode(led, gpio.INPUT)
上传 blink.lua 并立即执行：

$ sudo ./luatool.py -p /dev/ttyUSB0 -b 9600 -f blink.lua -d
8.2 远程控制 LED 闪烁
对于物联网来讲，远程控制很关键。咱们这里演示如何通过 Wifi 开启一个服务端口 8888 用于控制 LED，remote_led.lua：

-- 开启 Wifi 并获得 NodeMCU IP 地址
-- ssid 和 pwd 分别为自家路由器的 SSID 和访问密码
local ssid="SSID"
local pwd="password"
ip=wifi.sta.getip()
print(ip)
if not ip then
    wifi.setmode(wifi.STATION)
    wifi.sta.config(ssid,pwd)
    print(wifi.sta.getip())
end
-- 开启一个 8888 的端口
-- 并通过 node.input() 调用 Lua 解释器控制 LED
srv=net.createServer(net.TCP)
srv:listen(8888,function(conn)
    conn:on("receive",function(conn,payload)
    node.input("gpio.mode(0, gpio.OUTPUT)")
    node.input("gpio.write(0, gpio.LOW)")
    end)
end)
上传 Lua 程序到服务器执行：

$ sudo ./luatool.py -p /dev/ttyUSB0 -b 9600 -f remote_led.lua -d
查看 NodeMCU 获取的 IP 地址：

$ sudo minicom -D /dev/ttyUSB0
> print(wifi.sta.getip())
192.168.0.104	255.255.255.0	192.168.0.1
并测试：

$ sudo apt-get install lynx
$ lynx 192.168.0.104:8888
8.3 开启一个 Telnet 服务
先从 NodeMCU.com 下载该例子，telnetd.lua：

-- a simple telnet server
s=net.createServer(net.TCP,180)
s:listen(2323,function(c)
    function s_output(str)
      if(c~=nil)
        then c:send(str)
      end
    end
    node.output(s_output, 0)
    -- re-direct output to function s_ouput.
    c:on("receive",function(c,l)
      node.input(l)
      --like pcall(loadstring(l)), support multiple separate lines
    end)
    c:on("disconnection",function(c)
      node.output(nil)
      --unregist redirect output function, output goes to serial
    end)
    print("Welcome to NodeMCU world.")
end)
上传并执行：

$ sudo ./luatool.py -p /dev/ttyUSB1 -b 9600 -f telnetd.lua -d
通过 telnet 连接：

$ sudo apt-get install telnet
$ telnet 192.168.0.104 2323
Trying 192.168.0.104...
Connected to 192.168.0.104.
Escape character is '^]'.
Welcome to NodeMCU world.
> print('Hello, NodeMCU Telnet')
Hello, NodeMCU Telnet
>
有了 telnet 服务，咱就可以不依赖串口而是直接通过 Wifi 上传 Lua 脚本了：

$ cat test.lua
print('Upload via telnet service')
$ sudo ./luatool.py --ip 192.168.0.104:2323 -f test.lua -d -v

