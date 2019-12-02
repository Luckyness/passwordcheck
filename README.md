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
![](index_files/f2921a04-ded7-4a46-883a-c341bd23e5af.png)
![](index_files/9c70494b-55be-4a0f-97a7-7d9b7947151f.png)
# 3.源码修改
## 1.参考pg_cron的源码在配置文件内增加一个参数

```c
/* 引入扩展 */
#include "utils/guc.h"
……
……
/*
* 配置文件内passwordcheck.level='true' 为需要特殊字符
* passwordcheck.level='false' 为只需要英文和数字
*/
static  bool passwordcheck_level =  false;
……
……
void
_PG_init(void)
{
	/* 定义密码级别参数 */
    DefineCustomBoolVariable(
        "passwordcheck.level",
        gettext_noop("passwordcheck_level true: Password must contain leter, number, special characters;false : Password must contain leter, special characters"),
        NULL,
        &passwordcheck_level,
        false,
        PGC_POSTMASTER,
        GUC_SUPERUSER_ONLY,
        NULL, NULL, NULL);

	/* activate password checks when the module is loaded */
	check_password_hook = check_password;
}

```


## 2.修改源码配置校验数字

```c
        if(passwordcheck_level)
        {
            /* check if the password contains both letters and number and specialchar */
            pwd_has_number =  false;
            pwd_has_special =  false;
            pwd_has_letter =  false;
            for (i =  0; i < pwdlen; i++)
            {
                if (isalpha((unsigned  char) password[i]))
                    pwd_has_letter =  true;
                else  if (isdigit((unsigned  char) password[i]))
                    pwd_has_number =  true;
                else
                    pwd_has_special =  true;
            }
            if (!pwd_has_number ||  !pwd_has_letter ||  !pwd_has_special)
                ereport(ERROR,
                        (errcode(ERRCODE_INVALID_PARAMETER_VALUE),
                        errmsg("password must contain both letters and number and specialchar")));
        }
        else
        {
            /* check if the password contains both letters and non-letters */
            pwd_has_letter =  false;
            pwd_has_number =  false;
            for (i =  0; i < pwdlen; i++)
            {
                if (isalpha((unsigned  char) password[i]))
                    pwd_has_letter =  true;
                else
                    pwd_has_number =  true;
            }
            if (!pwd_has_letter ||  !pwd_has_number)
                ereport(ERROR,
                        (errcode(ERRCODE_INVALID_PARAMETER_VALUE),
                        errmsg("password must contain both letters and nonletters")));
        }
```
