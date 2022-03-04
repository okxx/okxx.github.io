---
layout: post
title: 软件环境自动构建脚本
categories: linux
tags: linux
sidebar: []
---

Software environment construction

 ### xx.sh
```bash
#!/bin/bash
#
# NOTE: 基于服务构建相关目录，用户，权限，数据库等相关脚本
#
#
# 用户:用户组
#       user:user_group
# 服务:服务ID
#       service:1000
# 目录:目录权限
#       user /home/boilerplate/ 755
# 数据库:
#       1.username：password
#       2.database
#
#############################
# 读取配置文件函数
#############################
conf_file=$1
ini(){
    conf_section=$2
    conf_option=$3
    RESULT=`awk -F '=' '/\['$conf_section'\]/{a=1}a==1&&$1~/'$conf_option'/{print $2;exit}' $conf_file`
    echo $RESULT``
}

######################################
# 用户相关
######################################
userBuild(){
    conf_section="os"

    os_user=$(ini $conf_file $conf_section "user")

    os_group=$(ini $conf_file $conf_section "group")
   
    egrep "^$os_group" /etc/group >& /dev/null
    if [ $? -ne 0 ]
    then
        groupadd $os_group
        echo "--- groupadd: ${os_group}"
    fi

    if id -u $os_user > /dev/null 2>&1; then
        echo "user：${os_user}  exists."
    else  
        useradd -g $os_group -m $os_user
        echo "--- useradd -g: ${os_user} | ${os_group}"
    fi
}

######################################
# 软件相关
######################################
softBuild(){
    conf_section="soft"

    # 提取软件目录地址
    soft_path=$(ini $conf_file $conf_section "path")
    echo "soft_path：${soft_path}"

    # 提取软件名称
    soft_name=$(ini $conf_file $conf_section "name")
    echo "soft_name: ${soft_name}"

    # 构建软件目录
    buildPath=$soft_path/$soft_name
    echo "buildPath: ${buildPath}"
    if [ ! -d $buildPath ]; then
        mkdir -p $buildPath
    fi 

    # 分配权限
    usermod -d ${soft_path} ${os_user}
    chown -R ${os_user}:${os_group} ${soft_path}/*
}


######################################
# 数据库相关
######################################
databaseBuild(){
    conf_section="db"

    db_name=$(ini $conf_file $conf_section "name")

    db_account=$(ini $conf_file $conf_section "account")

    db_password=$(ini $conf_file $conf_section "password")

    ruser="root"
    rpass="xxxxxxx"
    docker exec -it containerID bash -c "mysql -u ${ruser} -p${rpass} -s mysql -e\"
    create user if not exists '${db_account}'@'%' identified by '${db_password}';
    grant select,insert,update,delete,create on ${db_name}.* to ${db_account};
    \""
}


###################
# 执行函数:
#       执行顺序：user > soft  > database
###################
userBuild
softBuild
databaseBuild

echo "successful."
exit 0

```

#### xxx.conf：
```bash
[os]
user=xxx
group=xxx

[soft]
path=/home/boilerplate
name=xxx-service

[db]
name=xxx
account=xxx
password=xxx
```

#### 执行：
```bash
sh xx.sh xxx.conf
```
