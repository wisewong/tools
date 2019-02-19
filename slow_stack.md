# slow_stack

## 作用
查询机器上Tomcat服务CPU使用率最高的几个线程

## 脚本


```bash
#!/bin/sh
#changed MIao
  
PROG=`basename $0`
  
JAVA_HOME=你的java路径
WEB_HOME=你的web项目所在路径
  
  
jstack="$JAVA_HOME/bin/jstack"
  
usage() {
        cat <<EOF
Usage:
        ${PROG} <pid|webapp> [<count>]
        Find out the highest cpu consumed threads of java, and print the stack of these threads.
Example:
        ${PROG} twell 50
EOF
    exit $1
}
  
redEcho() {
    if [ -c /dev/stdout ] ; then
        # if stdout is console, turn on color output.
        echo -e "\033[1;31m$@\033[0m"
    else
        echo "$@"
    fi
}
  
  
printStackOfThreads() {
    while read threadLine ; do
  
        threadId=`echo ${threadLine} | awk '{print $1}'`
        threadId0x=`printf %x ${threadId}`
        pcpu=`echo ${threadLine} | awk '{print $2}'`
  
        jstackFile=/tmp/${uuid}_${javaid}
  
        [ ! -f "${jstackFile}" ] && {
            sudo -u tomcat $jstack ${javaid} > ${jstackFile} || {
                redEcho "Fail to jstack java process ${javaid}"
                rm ${jstackFile}
                continue
            }
        }
  
        redEcho "The stack of busy(${pcpu}%) thread(${threadId}/0x${threadId0x}) of java pid(${javaid}) all times($3):"
        sed "/nid=0x${threadId0x}/,/^$/p" -n ${jstackFile}
    done
  
}
  
if echo $1 | grep -q "$WEB_HOME"
then
                export CATALINA_BASE=$1
else
                export CATALINA_BASE=$WEB_HOME/$1
fi
  
if [ -e $CATALINA_BASE/conf/server.xml ]
then
                javaid=`ps aux |grep "java"|grep "Dcatalina.base=$CATALINA_BASE "|grep -v "grep"|awk '{ print $2}'`
else
                javaid=`ps -eo pid | grep "^$1$"`
fi
  
if [ -z "$javaid" ] ; then
        redEcho "process ($1) not found !"
        usage 1;
fi
  
if [ -z "$2" ] ; then
        count=1000
else
        (( count = $2 +7 ))
fi
  
uuid=`date +%s`_${RANDOM}_$$
  
top -p $javaid -H -n 1 -b | sed -n "8,${count}p" | awk '{print $1,$9,$11}' | printStackOfThreads
  
rm /tmp/${uuid}_*
```
替换上面脚本中的java环境变量以及web项目路径:
```
JAVA_HOME=你的java路径
WEB_HOME=你的web项目所在路径
```
## 使用

假设你的tomcat进程是用tomcat账户启动的,用法如下：
```bash
sudo chmod 755 ./slow_stack.sh
sudo -utomcat ./slow_stack.sh 进程pid  使用率最高的count个线程
```
