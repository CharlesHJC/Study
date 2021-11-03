# Ubuntu20.04下的环境配置
1. git的安装和使用
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
$git pull origim main
```

2. 使用rsa公钥验证github登陆
```bash
$ssh-keygen -b 8192 -C "charles_hjc@outlook.com" -t rsa
# 然后一直回车 就会在.ssh目录中生成id_rsa和id_rsa.pub两个文件.pub就是公钥
# 将.pub公钥文件内容copy到github->setting->SSH keys
# 下次使用git时将会验证github的用户名和邮箱，验证过后关联成功。
```