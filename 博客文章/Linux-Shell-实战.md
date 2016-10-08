---
title: Linux Shell 实战
date: 2016-10-03 16:11:54
tags: Linux
category: Linux
---


## monitor_man.sh
```shell
#!/bin/bash

resettem=$(tput sgr0)
declare -A ssharray
i=0
numbers=""

for script_file in `ls -I "monitor_man.sh" ./`
do
  echo -e "\e[1;35m" "The Script:" ${i} '==>' ${resettem} ${script_file}
  ssharray[$i]=${script_file}
  numbers="${numbers} | ${i}"

  i=$((i+1))
done

while true
do
  read -p "Please input a number [ ${numbers} ]:" execshell
  if [[ ! ${execshell} =~ ^[0-9]+ ]];then
        exit 0
  fi
done
```

## system_monitor.sh
```shell
#!/bin/bash

clear

if [[ $# -eq 0 ]]
then

# Define Variable reset_terminal
reset_terminal=$(tput sgr0)

# Check OS Type
os=$(uname -o)
echo -e '\E[32m' "Check OS Type" $reset_terminal $os

# Check OS Release Version and Name
os_name=$(cat /etc/issue|grep -e "Server")
echo -e '\E[32m' "Check OS Release Version and Name" $reset_terminal $os_name

# Check Architecture
architecture=$(uname -m)
echo -e '\E[32m' "Check Architecture" $reset_terminal $architecture

# Check Kernel Release
kernel_release=$(uname -r)
echo -e '\E[32m' "Check Kernel Release" $reset_terminal $kernel_release

# Check Hostname $HOSTNAME
host_name=$(uname -n)
echo -e '\E[32m' "Check Hostname" $reset_terminal $host_name

# Check Internal IP
internal_ip=$(hostname -I)
echo -e '\E[32m' "Check Internal IP" $reset_terminal $internal_ip

# Check External IP
external_ip=$(curl -s http://ipecho.net/plain)
echo -e '\E[32m' "Check External IP" $reset_terminal $external_ip

# Check DNS
name_servers=$(cat /etc/resolv.conf | grep -E "\<nameserver[ ]+" | awk '{print $NF}')
echo -e '\E[32m' "Check DNS" $reset_terminal $name_servers

# Check if connected to Internet or not
ping -c 2 baidu.com &> /dev/null && echo "Internet:Connected" || echo "Internet:Disconnected"
echo -e '\E[32m' "Check if connected to Internet or not" $reset_terminal $ping

# Check Logged In Users
who>/tmp/who
echo -e '\E[32m' "Logged In Users" && cat /tmp/who
rm -f /tmp/who

# Check System Memory Usages
system_men_usages=$(awk '/MemTotal/{total=$2}/MemFree/{free=$2}END{print (total-free)/1024}' /proc/meminfo)
apps_mem_usages=$(awk '/MemTotal/{total=$2}/MemFree/{free=$2}/^Cached/{cached=$2}/Buffers/{buffers=$2}END{print (total-free-cached-buffers)/1024}' /proc/meminfo)

echo -e '\E[32m' "System Memuserages" $reset_terminal $system_men_usages
echo -e '\E[32m' "Apps Memuserages" $reset_terminal $apps_mem_usages

load_average=$(top -n 1 -b | grep "load average:" | awk '{print $12 $13 $14}')
echo -e '\E[32m' "load averages" $reset_terminal $load_average

disk_average=$(df -hP | grep -vE 'Filesystem|tmpfs' | awk '{print $1 " " $5}')
echo -e '\E[32m' "disk averages" $reset_terminal $disk_average

fi
```

## check_server_sh
```shell
#!/bin/bash

resettem=$(tput sgr0)

nginx_server='http://10.170.22.217'

check_nginx_server()
{
	status_code=curl -m 5 -s -w %{http_code} ${nginx_server} -o /dev/null

	if [ $status_code -eq 000 -o $status_code -ge 500 ];then
		echo -e '\E[32m' "Check http server error! Response status code is " $resettem $status_code
	else
		http_content=$(curl -s ${nginx_server})
		echo -e '\E[32m' "Check http server ok! \n" $resettem $http_content
	fi
}

check_nginx_server
```
