[TOC]
因数据库入网检测须修改密码级别，在源有的passwordcheck插件上进行二次修改
添加配置参数为3的时候为密码内需要特殊字符
# 1.使用方式
* 替换目录 ../postgresql-11.4/contrib/passwordcheck 下的 passwordcheck.c
* 编译安装 make && make install
* postgresql配置文件内修改 (postgresql.conf)
* shared_preload_libraries = 'passwordcheck'
* passwordcheck.level = 'true'
![](index_files/1abfaa30-e518-48cf-89f4-5bc8ef22df34.png)

# 2.效果
当密码长度足够，不符合规则的时候，无法新建用户
