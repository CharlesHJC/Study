# Ubuntu20.04下的环境配置
1. 安装git
```bash
#git安装
$sudo apt-get update
$sudo apt-get upgrade & yes
$sudo apt-get install git
$git --version

#git配置
$git config --global user.name CharlesHJC
$git config --global user.email charles_hjc@outlook.com

#查看当前git缓存空间状态 
$git status

#git使用 clone，push，pull
$git clone https://github.com/CharlesHJC/Study.git
$git add .
$git commit -m "2021/10/20"
$git push origin main
```