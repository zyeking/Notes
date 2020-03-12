# 	1. 自动创建数据时间，不受其他修改影响的`NOW()`

1. 在`选项`下的`公式`启用`迭代计算`使得excel不会报循环引用的警告⚠

2. 此处的`C列`会自动根据`D列`数据创建时间，该时间不会根据其他的列改变

```
 excel =IF(D2="","",IF(C2="",NOW(),C2))
```

   
![](https://i.loli.net/2019/10/17/gZhRXkd2zKNyvmn.jpg)

   

参考URL：

+ [循环引用](https://jingyan.baidu.com/article/20b68a8851fc1e796dec6262.html)

+ [自动创建时间后，锁定时间](https://zhidao.baidu.com/question/94537286.html)



