---
title: "Contiki OS"
date: 2020-05-26T12:44:06+07:00
draft: false
---
Lưu ý bài hướng dẫn chỉ dành cho Tiến Phúc đọc mới hiểu.

Đầu tiên bạn phải cài đặt một số tool cần thiết cho ubuntu.

```console
sudo apt-get install git ant openjdk-8-jdk gcc-msp430 libncurses-dev net-tools gcc-arm-none-eabi python3-serial
```
Nếu bạn đã cài Java trước đã thì để configure lại, sử dụng lệnh sau
```console
sudo update-alternatives --config java
```

Sau đó tải [Contiki-OS](https://github.com/contiki-os/contiki) về.
```console
git clone https://github.com/contiki-os/contiki.git
```

Giờ chúng ta thử mở giả lập Cooja nhé, nếu lần đầu mở thì bạn phải setup MSPSim cho Cooja
```console
cd contiki/tools/cooja/
git submodule update --init
sudo ant run
```
Giờ chúng ta thử nhúng 1 node thành Z1 giả lập thử nha

Đầu tiên chúng ta phải ***make*** node ***border-router.z1*** thử nha

```console
cd contiki/examples/ipv6/rpl-border-router/
make TARGET=z1
```
Tạo 1 project mới trên cooja, sau đó add mote vào bằng cánh ***Motes>Add motes>Create new mote type>Z1 mote***
Sau đó chọn ***Browser>border-router.z1>Add motes***

Như vậy là ta đã thêm một mote vào Cooja, tiếp theo chúng ta sử dụng tunslip6 để kết nối mote ra Internet và Làm SERVER.

Để cái tunslip6, ta làm như sau:

```console
cd contiki/tools
make tunslip6
```
Để configure mote làm SERVER trên Cooja ta chọn ***Tool>Serial socket (SERVER)>Z1 1...*** sau đó modify ***Listen port 60001***  rồi ***Start***, Rồi ***Start*** trên ***Simulation control*** . Sau đó bật tunslip6

```console
sudo ./tunslip6 -a 127.0.0.1 aaaa::1/64
```
________________________________________
Bây giờ thử viết một virtual sensor nha:

đầu tiên ta cần tạo một folder demo-sensor chứa những file sau:
1. mysensor.h
2. mysensor.c
3. demo-sensor.c
4. Makefile

- ***mysensor.h***
```c
#ifndef MYSENSOR_H
#define MYSENSOR_H
struct Sensor {
char name[15];
float value;
};
struct Sensor read_temperature();
struct Sensor read_humidity();
#endif
```
- ***mysensor.c***
```c
#include "mysensor.h"
#include <string.h>
#include <stdlib.h>
float random_value(float min, float max)
{
float scale = rand() / (float) RAND_MAX;
return min + scale * (max - min);
}
struct Sensor read_temperature()
{
struct Sensor temp;
strncpy(temp.name, "Temperature", 15);
temp.value = random_value(0, 35);
return temp;
}
struct Sensor read_humidity()
{
struct Sensor humdidty;
strncpy(humdidty.name, "Humidity", 15);
humdidty.value = random_value(40, 80);
return humdidty;
}
```
- ***demo-sensor.c***
```c
// demo-sensor.c
#include "contiki.h"
#include "sys/etimer.h"
#include "mysensor.h"
#include <stdio.h>
PROCESS(sensor_process, "Sensor process");
AUTOSTART_PROCESSES(&sensor_process);
static struct etimer timer;
PROCESS_THREAD(sensor_process, ev, data)
{
PROCESS_BEGIN();
printf("Demo Virtual Sensor\n");
while(1) {
etimer_set(&timer, 3 * CLOCK_SECOND);
PROCESS_WAIT_EVENT_UNTIL(etimer_expired(&timer));
struct Sensor temp = read_temperature();
printf("%s: %.2f\n", temp.name, temp.value);
//printf("%f\n", temp.value);
struct Sensor hum = read_humidity();
printf("%s: %.2f\n", hum.name, hum.value);
//printf("%f\n", hum.value);
printf("------------\n");
}
PROCESS_END();
}
```
- ***Makefile***
```console
CONTIKI_PROJECT = demo-sensor
all: $(CONTIKI_PROJECT)
PROJECT_SOURCEFILES += mysensor.c
CONTIKI = ../..
include $(CONTIKI)/Makefile.include
```

Bây giờ thử make file ***.native*** chạy thử hen:

```console
make TARGET=native
./demo-sensor.native
```
Kết quả sẽ hiện ra như này
![virtual-sensor](/img/2.jpg)

________________________________________
Nhấn lệnh dài quá nên mình nên sử dụng **Alias** ,một tool của Ubuntu giúp mình viết tắt lệnh.

Đầu tiên chúng ta tạo một bashsr để lưu các câu lệnh

```console
touch ~/.bashrc
```
Sau đó mở file mới tạo lên, ở đây mình sử dụng text editor mặc định của ubuntu là gedit, ngoài ra bạn có thể sử dụng vim, vscode, atom,...

```console
gedit ~/.bashrc
```

Bây giờ bạn có thể gắn bắt kì lênh bằng thích, ví dụ tạo câu lệch tắt cho tunslip6

```shell
alias tunslipcooja='sudo /home/thesi/contiki/tools/tunslip6 -a 127.0.0.1 aaaa::1/64'
```
***LƯU Ý*** bạn nên đổi lại ***username*** của Ubuntu của bạn, ở đây ***username*** mình là ***thesi***. Và ở đây minh sử dụng đại chỉ rõ ràng của tool ***/home/thesi/contiki/tools/tunslip6***. Nên câu lệnh này có thể bất kì ở đâu trên **Terminal** mà k cần phải **cd** đến đúng folder của nói.

Như vậy là mình tạo được một lệnh **tunslipcooja** ngắn gọn.

Nhưng bạn cần *lưu lại* và đăng kí với hệ điều hành mới sử dụng được nha. Để đăng kí bạn làm như sau:

```console
source ~/.bashrc
```

Bây giờ chạy thử lệnh tắt mình.
```console
tunslipcooja
```
Như vậy là bạn đã biết được cánh tạo một câu **lệnh tắt** bằng **alias** và cánh đăng kí của nó. Để show các câu lệnh tắt của mình đang có, bạn sử dụng lệnh sau:

```console
alias
```
Sau đây là vài lênh alias mình hay sử dụng trong Contiki
```console
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'
alias cdson='cd /home/thesi/contiki/examples/cc2538dk/00_sls'
alias cooja='cd /home/thesi/contiki/tools/cooja/;sudo ant run'
alias dump0='sudo /home/thesi/contiki/tools/sky/serialdump-linux -b115200 /dev/ttyUSB0'
alias dump1='sudo /home/thesi/contiki/tools/sky/serialdump-linux -b115200 /dev/ttyUSB1'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'
alias ls='ls --color=auto'
alias mk1310='make TARGET=srf06-cc26xx BOARD=launchpad/cc1310'
alias mk13xx='make TARGET=srf06-cc26xx BOARD=srf06/cc13xx'
alias mk2538='make TARGET=cc2538dk'
alias mksky='make TARGET=sky'
alias prog0='sudo python3 /home/thesi/contiki/tools/cc2538-bsl/cc2538-bsl.py -e -w -v -p /dev/ttyUSB0 -a 0x00200000'
alias prog1='sudo python3 /home/thesi/contiki/tools/cc2538-bsl/cc2538-bsl.py -e -w -v -p /dev/ttyUSB1 -a 0x00200000'
alias pty='sudo putty -serial -sercfg 115200,8,n,1,N'
alias tunslip0='sudo /home/thesi/contiki/tools/tunslip6 -B 115200 -s /dev/ttyUSB0 aaaa::1/64'
alias tunslip1='sudo /home/thesi/contiki/tools/tunslip6 -B 115200 -s /dev/ttyUSB1 aaaa::1/64'
alias tunslipcooja='sudo /home/thesi/contiki/tools/tunslip6 -a 127.0.0.1 aaaa::1/64'
```
***Các bạn nhớ thấy tên ubuntu của mình lại nha, của mình ở đây "thesi"***
________________________________________

```console
cd contiki/example/cc2538dk
make TARGET=2538dk # or mk2538 (lệnh này đã được cài trong alias rồi nha)
prog0 prog0 cc2538-demo.bin # Nhớ nhấn 2 nút Reset + Select cùng lúc để nạp code nha
```
________________________________________
