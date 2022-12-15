# Error Record

## Error 1

![](../MdImages/C%2B%2B/error1.png)

 以上是编译时报错的信息，出错的语句是：
```C++
cout << (int)arr << endl;
```
### 原因为：
![](../MdImages/C%2B%2B/solu1.png)

### 解决办法为：

```C++
cout << (long int)arr << endl;
```