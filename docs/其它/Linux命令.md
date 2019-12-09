### 一、常用命令
##### 1、删除文件
rm -rf /data/elk/tomcat1
删除文件夹下以某个字母开头的所有文件
find ./ -name 'news_video_vec_20180*' -exec rm {} \;

##### 2、移动文件
进入到文件夹所在目录：
mv elasticsearch-5.2.2 /data/elk/
文件夹重命名
mv elasticsearch-5.2.2 elasticsearch

##### 3、解压软件包
1.压缩命令：
    例子：把/xahot文件夹打包后生成一个/home/xahot.tar.gz的文件。
    tar -zcvf /home/xahot.tar.gz /xahot
2.解压缩命令：
    命令格式：tar  -zxvf   压缩文件名.tar.gz
    解压缩后的文件只能放在当前的目录。
    tar -xzf apache-tomcat-7.0.2.tar.gz

##### 4、查看端口使用情况
1.查找被占用的端口
netstat -tln  
netstat -tln | grep 8083  
 netstat -tln 查看端口使用情况，而netstat -tln | grep 8083 则是只查看端口8083的使用情况

2.查看端口属于哪个程序？端口被哪个进程占用
lsof -i :8083  

3.杀掉占用端口的进程
kill -9 进程id  

##### 5、查看当前启动的程序
jps

##### 6、创建文件
1.创建文件夹
mkdir workspace
2. 创建文件
vim a.txt
或者touch a.txt

##### 7、在文件后面追加内容
1.将 abc 追加到文件a.txt最后
echo "abc" >> a.txt
2. 将文件b.txt 中的内容追加到a.txt最后
cat b.txt >> a.txt

##### 8、跨服务器复制
1、在A服务器上操作，将B服务器上/home/lk/目录下所有的文件全部复制到本地的/root目录下，
命令为：scp -r root@43.224.34.73:/home/lk /root
2、 在A服务器上将/root/lk目录下所有的文件传输到B的/home/lk/cpfile目录下，
命令为：scp -r /root/lk root@43.224.34.73:/home/lk/cpfile

##### 9、查看7天前的文件
1、查看7天前的文件  
find ./ -type f -mtime +7
2、删除7天前的文件  
find ./ -type f -mtime +7 -exec rm -f {} \;

### 二、Linux权限方面
##### 1、修改目录所属用户
1、修改 tmp 目录所属用户为 root，用户组为 root
chown -R root:root /tmp
##### 2、给一个文件赋可执行权限
chmod 777 restart.sh
或
chmod u+x fusiondata_deploy.sh 

##### 3、防火墙
 1、查看想开的端口是否已开 
 	firewall-cmd --query-port=666/tcp 提示no表示未开
 2、开永久端口号 
 	firewall-cmd --add-port=666/tcp --permanent 提示 success 表示成功
 3、重新载入配置 
 	firewall-cmd --reload 比如添加规则之后，需要执行此命令
 4、再次查看想开的端口是否已开 
 	firewall-cmd --query-port=666/tcp 提示yes表示成功
 5、 若移除端口 
 	firewall-cmd --permanent --remove-port=666/tcp

### 三、查看日志方面
##### 1、查看最新日志信息
tail -100f info-2019-04-28-0.log 

##### 2、查看最早日志信息
cat  info-2019-04-28-0.log |more

##### 3、grep查询
（一）Grep OR 操作符
1.使用选项 -E
grep -E 选项可以用来扩展选项为正则表达式。 
grep -E 'pattern1|pattern2'

2.使用 egrep
egrep 命令等同于‘grep -E’。
egrep 'pattern1|pattern2' filename 

（二）Grep AND 操作

1.使用 -E 选项可以实现AND操作。
grep -E 'pattern1.*pattern2' filename  

2.使用多个grep命令
可以使用多个 grep 命令 ，由管道符分割，以此来实现 AND 语义。 
grep -E 'pattern1' filename | grep -E 'pattern2'  

（三）Grep NOT操作
1.使用选项 grep -v
使用 grep -v 可以实现 NOT 操作。
grep -v 'pattern1' filename  


### 四、Linux使用情况
##### 1、查看当前Linux版本
lsb_release -a 

##### 2、查看磁盘空间
df –h

##### 3、前文件夹下的磁盘使用情况
sudo du -sh *  或者 du --max-depth=1 -h 

##### 4、建立软连接
ln -s flowMonitor_new flowMonitor

##### 5、查看内存 
free

