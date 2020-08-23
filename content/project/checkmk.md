---
title: "Checkmk, Best-in-class infrastruture & application monitoring"
date: 2020-08-05T19:06:14+07:00
draft: false
authors: [Kien Pham]
tags: []
categories: []
---
So sánh 4 phiên bản của [Checkmk](https://checkmk.com/editions.html)

Tải checkmk theo OS của bạn [Checkmk download](https://checkmk.com/download.php?edition=dcee&version=stable). Bạn có thể tham khảo các phiên bản ở trên trước khi tải nhé.\
>Checkmk có thể chạy trên các OS:
>- Linux
>- Docker
>- VMware ESXi / VirturalBox

Như mình sử dụng Ubuntu 64bits, nên mình sẽ install như sau:

>1. Đầu tiên phải tải checkmk trên website về, thường là đuôi .deb
>2. Cài đặt file vừa tải về bằng **sudo apt-get install**
```console
sudo apt-get install gdebi-core
sudo apt-get install ./<checkmk.deb>
```
Để kiểm tra đã cái đặt thành công bạn chỉ cần kiểm tra version của nó

```console
$ omd version
OMD - Open Monitoring Distribution Version 1.6.0p14.cee.demo
```
Vậy là tải thành công, tiếp theo chúng ta tạo site để monitoring. Checkmk có hỗ trợ GUI nên rất dễ sử dụng, ngoài ra còn có thể dùng Web-API để monitor bằng CLI.

>Tiếp theo mình sẽ show các lệnh của **omd** có gì nhé

```console
$ omd help
omd help                        Show general help
omd version    [SITE]           Show version of OMD
omd versions                    List installed OMD versions
omd sites                       Show list of sites
omd update                      Update site to other version of OMD
omd start      [SERVICE]        Start services of one or all sites
omd stop       [SERVICE]        Stop services of site(s)
omd restart    [SERVICE]        Restart services of site(s)
omd reload     [SERVICE]        Reload services of site(s)
omd status     [SERVICE]        Show status of services of site(s)
omd config     ...              Show and set site configuration parameters
omd diff       ([RELBASE])      Shows differences compared to the original version files
omd umount                      Umount ramdisk volumes of site(s)
omd backup     [SITE] [-|ARCHIVE_PATH] Create a backup tarball of a site, writing it to a file or stdout
omd restore    [SITE] [-|ARCHIVE_PATH] Restores the backup of a site to an existing site or creates a new site
```

Nếu bạn là người sử dung **Linux** thông thảo thì k xa lạ gì các lệnh serivce (start, restart, stop, reload và status). Ngoài ra còn còn **omd diff** giống chức năng **diff** của linux, dùng để so sánh sự khác nhau của 2 files. Còn **omd backup** và **omd restore** như tên của nó với mục định backup.

>Bây giờ tạo thử 1 site monitor của checkmk nhé

```console
sudo omd site mytest
Adding /opt/omd/sites/mytest/tmp to /etc/fstab.
Creating temporary filesystem /omd/sites/mytest/tmp...OK
Restarting Apache...OK
Created new site mytest with version 1.6.0p14.cee.demo.

  The site can be started with omd start mytest.
  The default web UI is available at http://ubuntu/mytest/

  The admin user for the web applications is cmkadmin with password: tqb5Uh7D
  (It can be changed with 'htpasswd -m ~/etc/htpasswd cmkadmin' as site user.)
  Please do a su - mytest for administration of this site.
```

Như vậy bằng đã tạo được một site với **username: cmkadmin** và **password: tqb5Uh7D**
Để thấy đổi password thì bằng sử dụng

```console
sudo htpasswd -m ~/etc/htpasswd cmkadmin
```

Để đăng nhập vào site trên CLI thì bản sử dung

```console
su -mytest
```

Để sử dụng Web GUI thì bản nhớ phải start site đó lên nhé

```console
sudo omd start mytest
Starting mkeventd...OK
Starting liveproxyd...OK
Starting mknotifyd...OK
Starting rrdcached...OK
Starting cmc...OK
Starting apache...OK
Starting dcd...OK
Initializing Crontab...OK
```

Để vào Web GUI bằng cần biết IP của máy bạn, nếu không thì sử dụng localhost. Bạn nhập vào brower của bạn như sao **<yourIP>/mytest** hoặc **localhost/mytest**. Sau đó bạn đăng nhập bằng username password lúc mới tạo. Đăng nhập vào bản sẽ thấy màn hình của Checkmk

![WedGUI_checkmk!](/img/checkmk/webGUI.png "Default website checkMK")

Có nhiều kiểu để kết nói thiết bị, OS với checkmk thông qua:
1. Agent Bakery
2. SNMP (v1, v2c, v3)
3. Các datasource programs (via REST API, via vSphere, via Web Interface, etc.)

Mỗi kiểu cần các yêu cầu khác nhau. Đầu tiền là các **Agent Bakery**. Bạn có thể tải Agent ở **Monitoring Agents** trong tab **WATO - CONFIGURATION**
![Monitoring Agents!](/img/checkmk/Monitoring-Agents.png "Monitoring Agents")

**Agent Bakery** hỗ trợ các hệ OS như là: **AIX, Linux, Solaris, Windows**. Các bạn chỉ cần tải Agent vào OS cần monitor và install là xong rồi. Giả sử bây giờ mình monitor chính Ubuntu của mình luôn, thì mình sẽ tải agent về và install nó trên Ubuntu, mình sẽ tải file **.deb** nhé. Xong cài đặt file **.deb** giống cài checkmk ở trên thôi.
```console
sudo apt-get install ./check-mk-agent_1.6.0p14-080f89d2aa3762d2_all.deb
```
