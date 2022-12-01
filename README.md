c1

[Unit]
Description=My Shell Script

[Service]
ExecStart=/usr/bin/script.sh

[Install]
WantedBy=multi-user.target



=========================================


Lvmcntrl

#!/usr/bin/env bash
set -Eeuo pipefail
trap cleanup SIGINT SIGTERM ERR EXIT
script_dir=$(cd "$(dirname "${BASH_SOURCE[0]}")" &>/dev/null && pwd -P)

usage() {
cat«EOF
Usage: $(basename "${BASH_SOURCE[0]}") [-h] [-v] [-f] -p param_value arg1 [arg2...]
Скрипт предназначен для настройки lvm.
Команды для работы:
-h, —help
-p arg1 arg2, инициализирует тома для lvm c названиями arg1 и arg2
-v arg1 arg2 arg3, создает группу разделов arg1 lvm по разделам arg2 и arg3
-c arg1 agr2, создает логический том lvm размером arg1 блоков и названием arg2
-e arg1 agr2, изменяет логический том lvm arg2 на arg1 количество МБ
-r arg1, удаляет логический том arg1
EOF
exit

}

cleanup() {
trap - SIGINT SIGTERM ERR EXIT
# script cleanup here
}

setup_colors() {
if [[ -t 2 ]] && [[ -z "${NO_COLOR-}" ]] && [[ "${TERM-}" != "dumb" ]]; then
NOFORMAT='\033[0m' RED='\033[0;31m' GREEN='\033[0;32m' ORANGE='\033[0;33m'
BLUE='\033[0;34m' PURPLE='\033[0;35m' CYAN='\033[0;36m' YELLOW='\033[1;33m'
else
NOFORMAT='' RED='' GREEN='' ORANGE='' BLUE='' PURPLE='' CYAN='' YELLOW=''
fi

}


msg() {
echo >&2 -e "${1-}"

}


die() {
local msg=$1
local code=${2-1} # default exit status 1
msg "$msg"
exit "$code"
}

setmsg() {
msg "${CYAN}Message!${NOFORMAT}"
}

create(){
sudo lvcreate -l ${1} -n ${2} vg1
exit "1"
}

extend(){
sudo lvextend -L +${1}M /dev/vg1/${2}
exit "1"
}

remove(){
#gs="${2-}"
sudo lvremove /dev/vg1/${1}
exit "1"
}

vcreate(){
sudo vgcreate ${1} /dev/${2} /dev/${3}
exit "1"
}

pcreate(){
sudo pvcreate /dev/${1} /dev/${2}
exit  “1”
}

parse_params() {
# default values of variables set from params
flag=0
param="${@}"

while :; do
case "$1" in
-h | —help) usage ;;
-v | —vcreate) vcreate "${2-}" "${3-}" "${4-}" ;;
-p | —pcreate) pcreate "${2-}" "${3-}";;
-r | —remove) remove "${2-}" ;;
-c | —create) create "${2-}" "${3-}";;
-e | —extend) extend "${2-}" "${3-}"
param="${2-}"
shift
;;
-?*) die "Unknown option: $1" ;;
*) break ;;
esac
shift
done
args=("$@")

msg "${@}"

# check required params and arguments
[[ -z "${param-}" ]] && die "Missing required parameter: param"
[[ ${#args[@]} -eq 0 ]] && die "Missing script arguments"
return 0
}
setup_colors
parse_params "$@"
#setup_colors


========================================



Script

#!/bin/bash
addr=$1
nm=$2
gt=$3
dns=$4
add=$6
ad=$6

num1(){
file=etc/network/interfaces

sed -i 's/address.*/address\ '$addr'/' $file
sed -i 's/netmask.*/netmask\ '$nm'/' $file
sed -i 's/gateway.*/gateway\ '$gt'/' $file
sed -i 's/dns-nameservers.*/dns-nameservers\ '$dns'/' $file
}

usage(){
echo "This script must be run with parametr and argument."
echo "Usage: -p [argument]"
echo "for 1: address netmask gateway dns-nameservers"
echo "for 2: address"
}

num2(){
file=etc/network/iterfaces

sed -i 's/add.*/add\ '$add'/' $file
sed -i 's/via.*/via\ '$add'/' $file
}

num3(){
sudo nmcli conn add type bridge con-name br0 ifname br0
sudo nmcli conn add type ethernet slave-type bridge con-name bridge-br0 ifname enp24s1 master br0
sudo nmcli conn show —active
sudo nmcli conn up br0
}

#number 2
if [ $# -eq 6 ]
then
num1
num2
exit 0
fi

if [ $# -eq 2 ]
then
num3
exit 0
fi

#number 5
if [ $# -le 1 ]
then
usage
exit 1
fi

#number 5
if [[ ( $# == "--help") || $# == "-h" ]]
then
usage
exit 0
fi
