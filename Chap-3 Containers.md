入职以来代码主要使用string/vector/map 等容器进行操作，一些书中提到的小点summary:
 # String
- **In Cpp, string is a container, while in c string is just char array.**<br>
 
 Cpp中，string被定义为一种容器，但是在c中，string仅仅是char 数组。c_str() 这个方法可以将容器转换为char数组。C++中多用String容器，char数组的问题在于分配的内存单元没有被初始化的话，默认是garbage info，这种操作很危险，而cpp的容器类string 默认初始化为空。<br>
 
- 直接赋值： `string s(10, 'c'); // s is cccccccccc`
 
- `cin >> s` vs. `getline(cin, s)`: <br>

`cin`的操作符 `>>` 会摒弃所有的前导空格，并且以第一个空格出现结束。 <br>

```
// cin string is "    Hello World!"
string s1, s2;
cin >> s1 >> s2; // s1 is "Hello", s2 is "World!"
cout << s1 << a2 << endl; // result is "HelloWorld!"
```
所以s1是Hello，s2是World!，没有了所有的前导和中间空格。<br>

`getline()`则不会摒弃前导空格和中间空格，一直读到换行符为止(换行符不会被写入string)，即使一开始是换行字符，那么这个写进来的字符串为空。<br>

- 读取整个文件的string:

```
// cin >> version
int main()
{
    string word;
    while (cin >> word)
        cout << word << endl;
    return 0;
}

// getline() version
int main()
{
    string line;
    while(getline(cin, line))
        cout << line << endl;
    return 0;
}
```
- `size()` 返回`string::size_type`:

这个方法返回的值是一个`unsigned type`, 与`signed type`的`int`相比较要格外小心(因为int为负时unsigned值一定非常大)。不要笼统的认为这个方法返回的就是一个int,实际上是`string::size_type`这个companion type。引入这种companion type主要是为了编译出machine independent的代码。(没理解，难道int不是不同的机器所占字节数不一样吗？？？迷) 更迷的是每次我代码`int size = string.size()`都可以编译通过....

- string的比较原则：
如果两个string的长度不同，并且短的那个字符串每个字符都和长的相同，则短的比长的小。如果随机两个string做比较，则由第一个不同的char比较得出大小。

- string literals: 两个字符串string直接相加会报错，也就是说：
```
string str = "this is " + " not correct";
```
这种赋值是错误的。

- 使用`[]`访问单个字符，里面的数字大于0且小于`size()`,这个数字是`string::size_type`类型的。如果对不在范围内的字符进行访问，the behavior is undefined。



  
 
