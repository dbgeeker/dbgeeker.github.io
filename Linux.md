# Linux 配置

### 一、screenFetch

#### 1、下载

```bash
wget -O screenfetch-dev https://git.io/vaHfR
# 或者 git clone https://github.com/KittyKatt/screenFetch.git
# Debian系的可以用 apt install screenfetch
```

#### 2、重命名

```bash
mv screenfetch-dev screenfetch
```

#### 3、添加执行权限

```bash
chmod +x screenfetch
```

#### 4、登录时显示系统信息

把 **screenFetch** 放到 **PATH** 可查找到的路径下，编辑 **~/.bashrc** 文件，加入以下脚本。

```bash
if [ -f /usr/bin/screenfetch ]; then
    screenfetch
fi
```

> 参考：
>
> https://my.oschina.net/terwergreen/blog/1845498
>
> [https://chikorita.fun/2020/03/cheat-sheet/centos-%E5%AE%89%E8%A3%85screenfetch/](https://chikorita.fun/2020/03/cheat-sheet/centos-安装screenfetch/)
>
> https://www.howtoing.com/screenfetch-system-information-generator-for-linux
>
> https://linux.cn/article-6510-1.html
>
> https://my.oschina.net/ziluoxingjun/blog/1806165



### 二、终端提示符

**PS1** 是用来设置命令提示符的环境变量，可以在终端输入以下命令来查看当前的设置。

```bash
echo $PS1
```

**CentOS** 上的 **PS1** 变量默认是：

```bash
[\u@\h \W]\$ 
```

这样的命令提示不美观，而且当我们输入的 **Linux** 命令得到很多输出的时候我们很难找到命令提示符在哪里，所以可以通过设置 **PS1** 来改善命令提示符。

命令提示符是由一系列符号组合而成的，这些符号包括：

| 符号 | 意义                                                   |
| :--: | :----------------------------------------------------- |
|  \d  | 代表日期，格式为weekday month date，例如："Mon Aug 1"  |
|  \H  | 完整的主机名称                                         |
|  \h  | 仅取主机的第一个名字                                   |
|  \t  | 显示时间为24小时格式，如：HH:MM:SS                     |
|  \T  | 显示时间为12小时格式                                   |
|  \A  | 显示时间为24小时格式：HH:MM                            |
|  \u  | 当前用户的账号名称                                     |
|  \v  | BASH的版本信息                                         |
|  \w  | 完整的工作目录名称。家目录会以 ~代替                   |
|  \W  | 利用basename取得工作目录名称，所以只会列出最后一个目录 |
|  \#  | 下达的第几个命令                                       |
|  \$  | 提示字符，如果是root时，提示符为：# ，普通用户则为：$  |

我们可以通过颜色代码来修饰上述这些组件，颜色代码的格式为\[\e[F;Bm\]，其中F表示字体的颜色，编号30~37，B表示背景的颜色，编号40~47。

颜色表如下：

| 字体代码 | 背景代码 |  颜色  |
| :------: | :------: | :----: |
|    30    |    40    |  黑色  |
|    31    |    41    |  红色  |
|    32    |    42    |  绿色  |
|    33    |    43    |  黄色  |
|    34    |    44    |  蓝色  |
|    35    |    45    | 紫红色 |
|    36    |    46    | 青蓝色 |
|    37    |    47    |  白色  |

代码表：

| 代码 |   意义    |
| :--: | :-------: |
|  0   |    OFF    |
|  1   | 高亮显示  |
|  4   | underline |
|  5   |   闪烁    |
|  7   | 反白显示  |
|  8   |  不可见   |

根据以上说明，我配置的PS1如下所示（我直接在 **~/.bashrc** 中进行配置的，这样每次打开终端都会出现配置的效果）：

```bash
export PS1='\[\e[36;48m\][\u@\h \W]\$ \[\e[0m\]'
```

说明：

1. 这里1m背景色是近似透明的。
2. 注意在$符号输出之后，我们还要重置颜色为透明，也就是\[\e[0m\]，这样你输入的命令就不会受之前颜色设置的影响。

> 参考：
>
> https://www.jianshu.com/p/0ad354929baf
>
> https://www.cnblogs.com/starspace/archive/2009/02/21/1395382.html



### 三、ssh 配置密钥登录

#### 1、生成密钥对

```bash
ssh-keygen
```

```bash
[root@host ~]$ ssh-keygen  <== 建立密钥对
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): <== 按 Enter
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): <== 输入密钥锁码，或直接按 Enter 留空
Enter same passphrase again: <== 再输入一遍密钥锁码
Your identification has been saved in /root/.ssh/id_rsa. <== 私钥
Your public key has been saved in /root/.ssh/id_rsa.pub. <== 公钥
The key fingerprint is:
0f:d3:e7:1a:1c:bd:5c:03:f1:19:f1:22:df:9b:cc:08 root@host
```

密钥锁码在使用私钥时必须输入，这样就可以保护私钥不被盗用。当然，也可以留空，实现无密码登录。

现在，在 root 用户的家目录中生成了一个 .ssh 的隐藏目录，内含两个密钥文件。id_rsa 为私钥，id_rsa.pub 为公钥。

#### 2、**在服务器上安装公钥**

##### 方法一：

```
[root@host ~]$ cd .ssh
[root@host .ssh]$ cat id_rsa.pub >> authorized_keys
```

如此便完成了公钥的安装。为了确保连接成功，请保证以下文件权限正确：

```
[root@host .ssh]$ chmod 600 authorized_keys
[root@host .ssh]$ chmod 700 ~/.ssh
```

##### 方法二：

```bash
# 复制公钥到无密码登录的服务器上, 22 端口改变可以使用下面的命令 
#ssh-copy-id -i ~/.ssh/id_rsa.pub "-p 10022 user@server"
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.15.241
```

#### 3、禁止密码登录

##### 修改 SSH 配置文件：

> 实际上禁用密码验证就行了

```bash
# 编辑 sshd_config 文件 
vi /etc/ssh/sshd_config

# 禁用密码验证 
PasswordAuthentication no
# 启用密钥验证 
RSAAuthentication yes
PubkeyAuthentication yes
# 指定公钥数据库文件 
AuthorsizedKeysFile .ssh/authorized_keys

sed -i "s/^PasswordAuthentication.*/PasswordAuthentication no/g" /etc/ssh/sshd_config
sed -i "s/^#RSAAuthentication.*/RSAAuthentication yes/g" /etc/ssh/sshd_config
sed -i "s/^#PubkeyAuthentication.*/PubkeyAuthentication yes/g" /etc/ssh/sshd_config
sed -i "s/^#AuthorizedKeysFile.*/AuthorizedKeysFile .ssh\/authorized_keys/g" /etc/ssh/sshd_config
```

#### 4、修改私钥 passphrase

```bash
ssh-keygen -f id_rsa -p
```


> 参考：
>
> https://hyjk2000.github.io/2012/03/16/how-to-set-up-ssh-keys/
>
> https://wsgzao.github.io/post/ssh/
>
> https://mozillazg.com/2013/01/linux-ssh-change-passphrase.html