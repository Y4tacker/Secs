# php_curl

## demo

![image-20221221105734289](/Users/y4tacker/Desktop/1.Project/Secs/杂七杂八的Trick/SSRF/php_curl/index/image-20221221105734289.png)

## solve

很简单，很多年前blackhat就曾提到过，他会解析@后面那个，命令行的curl与php比较类似

![image-20221221105906293](/Users/y4tacker/Desktop/1.Project/Secs/杂七杂八的Trick/SSRF/php_curl/index/image-20221221105906293.png)

因此最终构造前面被当成了路径，0刚好指向本地，其实相当于访问了http://0/flag

```
https@0/test.octagon.net/1.php/../../flag
```

