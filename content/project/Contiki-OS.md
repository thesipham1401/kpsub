---
title: "Contiki OS"
date: 2020-05-26T12:44:06+07:00
draft: false
---
Lưu ý bài hướng dẫn chỉ dành cho Tiến Phúc đọc mới hiểu.

Đầu tiên bạn phải cài đặt một số tool cần thiết cho ubuntu.

```console
thesi@ubuntu:~$ sudo apt-get install git ant openjdk-8-jdk gcc-msp430
```
Nếu bạn đã cài Java trước đã thì để configure lại, sử dụng lệnh sau
```console
thesi@ubuntu:~$ sudo update-alternatives --config java
```

Sau đó tải [Contiki-OS](https://github.com/contiki-os/contiki) về.
```console
thesi@ubuntu:~$ git clone https://github.com/contiki-os/contiki.git
```

Giờ chúng ta thử mở giả lập Cooja nhé, nếu lần đầu mở thì bạn phải setup MSPSim cho Cooja
```console
thesi@ubuntu:~$ cd contiki/tools/cooja/
thesi@ubuntu:~/contiki/tools/cooja$ git submodule update --init
thesi@ubuntu:~/contiki/tools/cooja$ sudo ant run
```
Giờ chúng ta thử nhúng 1 node thành Z1 giả lập thử nha

Đầu tiên chúng ta phải ***make*** node ***border-router.z1*** thử nha

```console
thesi@ubuntu:~$ cd contiki/examples/ipv6/rpl-border-router/
thesi@ubuntu:~/contiki/examples/ipv6/rpl-border-router$ make TARGET=z1
```
Tạo 1 project mới trên cooja, sau đó add mote vào bằng cánh ***Motes>Add motes>Create new mote type>Z1 mote***
Sau đó chọn ***Browser>border-router.z1>Add motes***

Như vậy là ta đã thêm một mote vào Cooja, tiếp theo chúng ta sử dụng tunslip6 để kết nối mote ra Internet và Làm SERVER.

Để cái tunslip6, ta làm như sau:

```console
thesi@ubuntu:~/contiki/examples/ipv6/rpl-border-router$ cd ../../../tools/
thesi@ubuntu:~/contiki/tools$ make tunslip6
```
Để configure mote làm SERVER trên Cooja ta chọn ***Tool>Serial socket (SERVER)>Z1 1...*** sau đó modify ***Listen port 60001***  rồi ***Start***, Rồi ***Start*** trên ***Simulation control*** . Sau đó bật tunslip6

```console
thesi@ubuntu:~ sudo ./tunslip6 -a 127.0.0.1 aaaa::1/64
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
thesi@ubuntu:~/contiki/examples/demo-sensor$ make TARGET=native
thesi@ubuntu:~/contiki/examples/demo-sensor$ ./demo-sensor.native
```
Kết quả sẽ hiện ra như này
![virtual-sensor](/img/2.jpg)

To be continue...
