# 常用shell命令

## 文本三剑客
- grep
- sed
- awk


## 查找/搜索
- find

## 管道与重定向
```shell
cmd > file            #输出重定向(覆盖)
cmd >> file           #输出重定向(追加)
cmd 2> errfile        #错误重定向
cmd &> all            #全部重定向 bash4+
exec 3<&-             #关闭fd 3
cmd1 | cmd2 | cmd3    #管道
cmd <<< eeooff
"context"
eeooff        # here-doc 结束时的eeooff必须顶格
xargs -I{} 聪明的{}    #把前面的输出当参数
```

## 进程与新号
- `kill -l` 列出全部信号
- `trap <command> <signal>` 脚本捕获新号

## 压缩解压
- `tar -czf xxx.tar.gz /dir` 打包压缩
- `tar -xzf xxx.tar.gz` 解压
- `zip -r xxx.zip /dir` zip压缩
- `unzip xxx.zip` zip解压

## 权限与用户
- chmod 755 file
- chmod -R 755 dir
- chown user:group -R dir #修改文件所有者
- whoami


## 网络与下载
- `curl -O <url>` 下载
- `wget <url>` 下载
- `ssh user@ip` 登录
- `scp file user@ip:/path` 远程复制
- netstat
- `telnet <ip> <port>`



## 系统监控
- top 实时进程
- df 磁盘
- du 当前目录占用空间
- free 内存
- uptime 负载
- uname 系统信息

## shell脚本

#### ${}
|用法|含义+示例|实际场景|
|----|----|----|
|`${var:-default}`|var 为空或未定义时，用 default（不赋值）`:${PORT:-8080}`|默认参数神器|
|`${var:=default}`|为空时用 default，并写入变量`: ${PORT:=8080}`|强制设置默认值|
|`${var:+value}`|var 有值时用 value，否则什么都不展开`: ${DEBUG:+ -x}`|开启调试才加 -x|
|`${var:?error_msg}`|为空时报错并退出脚本`: ${INPUT:? "请输入文件"}`|参数必填检查|
|`${!var}`|间接变量引用（var 里存的是另一个变量名）`a=100; b=a; echo ${!b}`->100|配置动态变量|
|`${!prefix*} / ${!prefix@}`|列出所有以 prefix 开头的变量名`echo ${!HOME*}`|调试时打印一组变量|
|`${var#pattern} / ${var##pattern}`|从左删：# 最短匹配，## 最长匹配`p=/usr/local/bin/gcc; echo ${p##*/}`->gcc|取文件名 / 去路径|
|`${var%pattern} / ${var%%pattern}`|从右删：% 最短，%% 最长`f=backup.tar.gz; echo ${f%.tar.*}`->backup|去后缀|
|`${var/pattern/repl}`|替换第一次`${PATH//:/ }`|路径转空格|
|`${var//pattern/repl}`|替换全部`${str// /-}`|全量替换|
|`${var/#pattern/repl}`|匹配开头才替换`${url/#http/https}`|强制 https|
|`${var/%pattern/repl}`|匹配结尾才替换`${file/%jpg/png}`|批量改扩展名|
|`${#var}`|变量长度`echo ${#PATH}`|判断字符串长度|
|`${var:offset}`|截取子串（从0开始）`ver=3.12.1; echo ${ver:2}`→ 12.1|取版本号中间段|
|`${var:offset:length}`|截取指定长度`${var: -4}`（注意前面要空格）取最后4位|取后缀、尾部IP等|
|`${var^} / ${var^^}`|首字母大写 / 全部大写`${var,} / ${var,,}` → 小写|标准化变量|
