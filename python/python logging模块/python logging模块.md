
![BV1gM7HzuEEJ-[01:15]](./images/55f40944-7781-41b5-b5c6-ddeb7fa9504c-1.png)  
logging主要的用途也是在控制台中打印信息，与`print`类似，但是功能更丰富。  
![BV1gM7HzuEEJ-[01:23]](./images/55f40944-7781-41b5-b5c6-ddeb7fa9504c-2.png)  
logging记录的信息分为五个级别。  

![BV1gM7HzuEEJ-[02:03]](./images/55f40944-7781-41b5-b5c6-ddeb7fa9504c-3.png)  
默认指打印warning之后级别的信息。  

`logging.basicConfig`这个类可以进行logging的各种设置。  
主要的几个参数：  
`format`可以用来控制打印内容的格式。  
`level`用来控制打印显示的最低级别。  
`datefmt`可以用来控制时间显示的格式。  
`filename`可以将logging的消息打印在log文件而非控制台。  
![BV1gM7HzuEEJ-[10:03]](./images/55f40944-7781-41b5-b5c6-ddeb7fa9504c-5.png)  
其中format参数中使用类似于字符串格式化的形式，将一些变量信息写入字符串，比如%(levelname)s就代表当前logging信息的等级。具体可以参看[这里](https://docs.python.org/zh-cn/3/library/logging.html#logrecord-attributes)  

比如可以将logging的信息写入指定文件：  
![BV1gM7HzuEEJ-[14:10]](./images/55f40944-7781-41b5-b5c6-ddeb7fa9504c-6.png)  
![BV1gM7HzuEEJ-[14:12]](./images/55f40944-7781-41b5-b5c6-ddeb7fa9504c-7.png)  

