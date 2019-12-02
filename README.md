# 1.使用方式
* 替换目录 ../postgresql-11.4/contrib/passwordcheck 下的 passwordcheck.c
* 编译安装 make && make install
* postgresql配置文件内修改 (postgresql.conf)
* shared_preload_libraries = 'passwordcheck'
* passwordcheck.level = 'true'
![](index_files/1abfaa30-e518-48cf-89f4-5bc8ef22df34.png)

# 2.效果
pg数据库用户密码必须包括数字英文和特殊字符
