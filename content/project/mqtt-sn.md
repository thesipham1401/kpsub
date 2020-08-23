---
title: "MQTT for Sensor network"
date: 2020-08-02T13:19:20+07:00
draft: false
authors: [Kien Pham]
tags: []
categories: []
---
  Bây giờ chúng ta thử mô phỏng Cooja truyền tin từ 1 node lên broker. Đầu tiên bản hãy tải code mqtt-sn về.
Code đây mình chỉ chỉnh sửa lại file ***maincore.c*** cho phù hợp với nhu cầu nhé.
  Dầu tiên bản tải code mqtt-sn. *lưu ý* tải về bỏ trong thư mục của contiki, bởi vì code này sử dụng thư viện của contiki để **make**
```console
git clone https://github.com/thesipham1401/mqtt-sn-contiki_example.git
```
Bây giờ mình thử chỉnh sửa code để truyên vitural node mà ở bài trước mình đã hướng dẫn tạo cho các bạn.
Đầu tiên các bạn phải copy thư viện hôm trước mình tạo vào trong folder của mqtt-sn. Các thư viên gồm: **mysensor.h**
Khi bạn thêm một thư viện có 2 việc bạn phải làm. Đầu tiên bạn phải gọi nó trong file ***maincore.c***, thứ 2 bạn phải gọi nó trong ***Makefile***
1. **maincore.c**
```c
#include "mysensor.h"
```
2. **Makefile**
```code
all: main_core
PROJECT_SOURCEFILES += mqtt_sn.c mysensor.c # Define cho contiki
WITH_UIP6=1
UIP_CONF_IPV6=1
CFLAGS+= -DUIP_CONF_IPV6_RPL
CFLAGS += -DPROJECT_CONF_H=\"project-conf.h\"
CFLAGS += -ffunction-sections
LDFLAGS += -Wl,--gc-sections,--undefined=_reset_vector__,--undefined=InterruptVectors,--undefined=_copy_data_init__,--undefined=_clear_bss_init__,--undefined=_end_of_init__
CONTIKI=../ # Define folder Contiki ở đâu, ở đây mình tải mqtt-sn nằm ở **contiki/mqtt-sn** Nên chỉ cần 1 lần  "../"
include $(CONTIKI)/Makefile.include
```

Giờ mình tiếp tục sửa file ***maincore.c*** để truyền temperature và humidity cho Broker.

**maincore.c**
```c
#include "contiki.h"
#include "lib/random.h"
#include "clock.h"
#include "sys/ctimer.h"
#include "net/ip/uip.h"
#include "net/ipv6/uip-ds6.h"
#include "mqtt_sn.h"
#include "dev/leds.h"
#include "net/rime/rime.h"
#include "net/ip/uip.h"
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include "mysensor.h"

static uint16_t udp_port = 1884;
static uint16_t keep_alive = 5;
static uint16_t broker_address[] = {0xaaaa, 0, 0, 0, 0, 0, 0, 0x1};
static struct   etimer time_poll;
// static uint16_t tick_process = 0;
static char     pub_test_temp[50];
static char     pub_test_hum[50];
static char     device_id[17];
static char     topic_hw[25];
static char     *topics_mqtt[] = {"/thesi/test/teperature",
                                  "/thesi/test/humidity"};
// static char     *will_topic = "/6lowpan_node/offline";
// static char     *will_message = "O dispositivo esta offline"
// This topics will run so much faster than others

mqtt_sn_con_t mqtt_sn_connection;

void mqtt_sn_callback(char *topic, char *message){
  printf("\nMessage received:");
  printf("\nTopic:%s Message:%s",topic,message);
}

void init_broker(void)
{
  char *all_topics[ss(topics_mqtt)+1];
  mqtt_sn_connection.client_id     = device_id;
  mqtt_sn_connection.udp_port      = udp_port; // 1884
  mqtt_sn_connection.ipv6_broker   = broker_address;
  mqtt_sn_connection.keep_alive    = keep_alive;
  //mqtt_sn_connection.will_topic    = will_topic;   // Configure as 0x00 if you don't want to use
  //mqtt_sn_connection.will_message  = will_message; // Configure as 0x00 if you don't want to use
  mqtt_sn_connection.will_topic    = 0x00;
  mqtt_sn_connection.will_message  = 0x00;

  mqtt_sn_init();
  size_t i;
  for(i=0;i<ss(topics_mqtt);i++)
    all_topics[i] = topics_mqtt[i];
  all_topics[i] = topic_hw;

  mqtt_sn_create_sck(mqtt_sn_connection,
                     all_topics,
                     ss(all_topics),
                     mqtt_sn_callback);
  mqtt_sn_sub(topic_hw,0);
}


//PROCESS(init_system_process, "[Contiki-OS] Initializing OS");
PROCESS(sensor_process, "Sensor process");
//AUTOSTART_PROCESSES(&init_system_process,&sensor_process);
AUTOSTART_PROCESSES(&sensor_process);

static struct etimer timer;

PROCESS_THREAD(sensor_process, ev, data)
{
PROCESS_BEGIN();
//  debug_os("Initializing the MQTT_SN_DEMO");

	init_broker();

  	etimer_set(&time_poll, CLOCK_SECOND);

while(1) {

	etimer_set(&timer, 3 * CLOCK_SECOND);
	PROCESS_WAIT_EVENT_UNTIL(etimer_expired(&timer));
	struct Sensor temp = read_temperature();
	struct Sensor hum = read_humidity();
	sprintf(pub_test_temp, "%s: %.2f\n", temp.name, temp.value);
	mqtt_sn_pub("/thesi/test/teperature",pub_test_temp,true,0);

	sprintf(pub_test_hum, "%s: %.2f\n", hum.name, hum.value);
	mqtt_sn_pub("/thesi/test/humidity",pub_test_hum,true,0);
// debug_os("State MQTT:%s",mqtt_sn_check_status_string());
//	if (etimer_expired(&time_poll))
//		etimer_reset(&time_poll);
}
PROCESS_END();
}
```

Bây giờ chung ta hãy setting cho broker_mqtts

```console
cd /tools/mosquitto.rsmb/rsmb/src
sudo ./broker_mqtts config.mqtt
```
***config.mqtt***
```
trace_output protocol
listener 1883 INADDR_ANY
listener 1884 INADDR_ANY mqtt_s
ipv6 true
```

Nếu báo lỗi socket or cannot bind port 1883, thì có thể có mosquitto broker đang chạy. Bạn chỉ cần kiểm tra và kill nó đi bằng cánh

```console
ps aux | grep broker
```
![sample1](/img/mqtts1.PNG)
Xong ta tiếp hành kill các broker đang chạy, ở đây ta thấy 2 broker ID 73767 và ID 73768

```console
sudo kill -9 73767
sudo kill -9 73768
```
Thử chạy lại broker bằng lệnh trên nhé, nếu không được thì reset lại lun :D

![sample2](/img/mqtts2.PNG)

Xong tiến hành mở cooja ra sử dụng tunslip cho broker (border-router), và client là "maincore.c" nhé

Sử dụng **mosquitto_sub -t "#" -v** để xem các tin public của tất cả ("#"), nếu bạn mún coi cụ thể topic nào đó chỉ cần thay đổi "#" thành topic cụ thể (vd "thesi/test/temperature")

Và đây chỉ là mô phỏng nhé :v, chỉ cần thay đổi code phù hợp với mục đích của bạn là đc. Peace
