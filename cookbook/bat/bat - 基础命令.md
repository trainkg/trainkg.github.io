## BAT 基础命令范例

```bash
echo off 

rem print execute parameters
echo %1
:: if 命令

if "%1"=="king" pause echo param is king
if "%1"=="zhuyy" goto print
:: errorlevel 获取上一次命令的DOS返回值
if errorlevel 1 echo last command execute error

:: 临时变量,线程级别
set username=%1
echo %username%

:: ping 命令 + 输出到指定文件 (>)
rem output
ping sz.tencent.com > a.txt

:: 追加文件写入 符号 (>>)
rem ping sz1.tencent.com >> a.txt

rem print text
:: @echo off
echo this is test

:: 判断是否存在某个文件
rem if syntax
if exist D:\cookbook\a.txt del D:\cookbook\a.txt

rem netstat syntax & search txt
:: find 这是一个搜索命令，用来在文件中搜索特定字符串
netstat -a -n > a.txt
type a.txt | find "3306" && echo "Congratulations! You have infected GLACIER! mysql server is running"

rem del syntax
del a.txt

:: remark (注释) rem 提示信息
rem tip
echo rem tip , this is tip syntax

:: @ 执行窗口中不显示它后面这一行的命令本身, 在控制台不再显示具体的命令内容
@echo this is test informations.

:: echo 是一个开关命令执行  `echo off`将关闭回显，它后面的所有命令都不显示命令本身

:: pause 当前线程等待

:: ':' & goto 标签和跳转

:: call 调用其他脚本
call a.cmd

:: for、set、shift
for /? > for.txt
:: FOR /F "usebackq delims==" %i IN (`set`) DO @echo %i , 分析SET命令输出流

set /? > set.txt
shift /? >shift.txt
:: SHIFT 管理执行命令参数 SHIFT /2 ,剔除最初的执行命令第二个参数

:: '|' 顺序执行, 比较重要的是属于同一个管道, 后一个命令可以访问前一个命令的输出

:: > >> , >会清除掉原有文件中的内容后把新的内容写入原文件，而>>只会另起一行追加新的内容到原文件中

:: <、>&、<& 这三个命令也是管道命令


:: & 命令按顺序执行，而不管是否有命令执行失败

:: &&  与&命令不同之处在于，它在从前往后依次执行被它连接的几个命令时会自动判断是否有某个命令执行出错，一旦发现出错后将不继续执行后面剩下的命令

:: || 利用这种方法在执行多条命令时，当遇到一个执行正确的命令就退出此命令组合

:: 

exit

:print 
echo common
echo -t username
:end

cmd
```

