# 正则表达式  
> 一直记不住、用不好正则表达式。本文主要为b站视频[30分钟正则表达式教程](https://www.bilibili.com/video/BV1fm411C7fq/?spm_id_from=333.337.search-card.all.click&vd_source=d82de55cfde970cdf86016bef2c6de4e)的笔记，主要根据个人遇到较多但总是记不住和不能理解的点进行选择性记录，可能会省略一些比较基础或者比较复杂的知识点。  

## 常见的表达
- []，表示字符集合，匹配方括号中任意一个字符
- [^]，其中的^表示取反，并且只有在方括号内，^才表示取反
- 预定义字符，比如\d表示匹配数字，但其实[0-9]也可以用来表示任意一个数字，预定义字符主要是提供了更加快捷和方便的方式。
  - .，任意字符
  - \d，任意数字
  - \D，非数字，等价于[^0-9]
  - \w，字母、数字、下划线，等价于[A-Za-z0-9_]
  - \W，与\w相反
  - \s，空白字符，空格、换行、Tab
  - \S，任意非空白字符
  - \b，表示单词的边界，比如`\bin`就是匹配以`in`开头的单词里面的`in`。
- 量词
  - +，表示1或者多次
  - *，表示0或者多次
  - ？表示0或者1次
  - {n}，表示n次
  - {n,m}，表示n到m次，并且由于贪婪匹配，所以如果有m次就会将m个字符都匹配（虽然匹配n个字符也满足该正则表达式）
  - {n,m}?，表示n到m次，但是是非贪婪匹配（**？除了表示0次或者1次，也表示非贪婪匹配**）
- 分组
  - 分组就是使用()将内容括起来，可以将多个字符作为单独的一个整体。比如`ab+`不能匹配`abab`，而需要使用`(ab)+`才可以
  - 分组一个重要的作用是将内容进行捕获  
    举个例子，将`2015-05-02`从一段字符串中匹配出来，可以用`\d{4}-\d{1,2}-\d{1,2}`匹配出来，但是如果想要使用搜索到的字符串，则需要将其用()捕获，即`(\d{4})-(\d{1,2})-(\d{1,2})`。这样在对应不同的平台下就有不同的方法获取匹配的字符，比如在powershell脚本语言中，可以使用`.Group[1]`来获取第一个分组的内容，在本例中就是`2015`。
  - 分组的另一个作用就是引用。
    - 比如在单词`alibaba、tencent、google`中想要捕获第一个字母和最后一个字母相同的单词。就可以使用`\b([a-z])[a-z]*\1\b`，其中的`\1`就是引用了第一个分组。  
  - 分组的再一个用处就是可以进行前瞻和后顾匹配
    - 正向前瞻就是在()分组的前部加入`?=`，即(?=)这样就会匹配到整个正则表达式匹配中这个分组前面的内容，不会把分组的内容包括进去（但是在查找的时候是考虑了分组的，不是忽略该分组的意思）  
    - 负向前瞻就是在()分组的前部加入`?!`，其作用就类似于[^]的取反的作用
    - 后顾就是匹配到字符后面的内容（不包括分组内的字符本身），`(?<=)`表示正向后顾，`(?<！)`将=改成!就表示负向后顾。