
几天出现过一个bug，很隐蔽。记录一下。

```c++
uint64_t lastTime; // 毫秒
uin32_t nowTime = time(); 
if (lastTime <  (uint64_t)(nowTime*1000)){ // bug

}

if (lastTime <  (uint64_t)nowTime*1000){ // ok

}


```
这个逻辑判断是有问题的。nowTime是32位 *1000后可能就溢出了，然后再转成64位，结果已经被截断了。