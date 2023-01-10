# addslashes与preg_replace

遇到个有意思的案例，首先来看这样一串代码

![image-20230110153620439](/Users/y4tacker/Desktop/1.Project/Secs/杂七杂八的Trick/SQLI/addslashes与preg_replace/index/image-20230110153620439.png)

乍一看因为有addslashes，只要我们输入了单引号就会被转义，那有没有办法可以逃逸出这个单引号呢？

在我们preg_replcae函数第二位参数当中其实也是有一些保留字符(可以在php源码当中找到)

![image-20230110154731830](/Users/y4tacker/Desktop/1.Project/Secs/杂七杂八的Trick/SQLI/addslashes与preg_replace/index/image-20230110154731830.png)

$可以用来获取对应分组去做替换，而`\`只有一个用法，它只能被用来转义`\`字符，也就是`\\`=>`\`

因此配合这个特效结合起来我们便可以构造出

`1\' and 1=1#`

经过addslashes处理后=>`1\\\' and 1=1#`

再经过preg_replace处理后=>`1\\' and 1=1#`(`\\`=>`\`)

此时我们便成功逃逸出了这个单引号

![image-20230110155426224](/Users/y4tacker/Desktop/1.Project/Secs/杂七杂八的Trick/SQLI/addslashes与preg_replace/index/image-20230110155426224.png)

