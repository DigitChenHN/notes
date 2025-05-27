# [10分钟讲清楚 Prompt, Agent, MCP 是什么](https://www.bilibili.com/video/BV1aeLqzUE6L?spm_id_from=333.788.videopod.sections&vd_source=d82de55cfde970cdf86016bef2c6de4e)  
## LLM的本质
![BV1aeLqzUE6L-[02:22]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-1.png)  
目前（2024-5-10）语言大模型的本质还是输入文本然后输出文本（文本的概率）。    

## 如何让llm使用工具自行完成任务  
![BV1aeLqzUE6L-[03:50]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-3.png)

开源项目autogpt。  
首先基于各种编程语言实现一些函数。  
然后将这些函数注册到AutoGPT中。这个注册的过程很有可能就是将函数的描述、输入输出、实现的功能等信息放在后续与ai的对话的system prompt中。  
除此之外将函数注册到AutoGPT之后，Autogpt还将能够解析llm返回的内容，并且为其调用工具（这里的实现还挺难的？）。  
调用工具之后得到结果，AutoGPT将返回结果再次以prompt的形式输入到llm中。  
这种在用户、llm、工具之间起到传话的作用的工具就是agent。  
![BV1aeLqzUE6L-[04:20]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-4.png)  

## 解析llm返回的文本并且按照文本提示调用工具  
实现agent比较难的一个点就在于如何解析llm返回的文本？  
首先可以考虑在system prompt中规定了llm要输出的结构，要求其结构化输出之后，就可以方便地解析llm的输出了。  
但是由于llm本质上是根据概率输出文本，所以有可能输出的文本还是不符合预设的结构，那么agent需要具备的第二个功能就是处理这种错误的情况。 
![BV1aeLqzUE6L-[04:41]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-5.png)  
一种解决方案就是agent软件可以发现llm的输出不符合预设的格式，那么直接进行重试。比如cline这个agent软件。  
但是大模型往往输入相同时，输出也是相同的。重试的方法似乎并不靠谱。  

## function calling
一般这个功能直接由大模型厂商提供。  

![BV1aeLqzUE6L-[05:12]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-6.png)

function calling本质上是对上述👆的system prompt中对于函数的描述，用更加标准和规范的结构展示。比如：  
```json
{
  "name": "list_files",
  "desc": "列目录",
  "params": {
    "path": "str"
  }
}  
```
![BV1aeLqzUE6L-[06:08]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-7.png)  
llm的回复也进行了格式化。对于格式化的回复，agent这个程序进行解析并且调用函数就非常容易且准确。  
如果llm的回复不是以上的格式，llm服务端自行就会检测到，从而直接进行重试，而不需要客户端进行处理和反应。  
因此function calling主要结构化两个文本，一个是agent对于函数的描述，一个是llm的输出。  

## MCP（Model Context Protocol）
由于function calling是由各个llm服务商提供的，因此提供的api不同（应该可以理解为描述函数，以及ai调用函数的输出的格式不同？）；此外很多开源模型也不支持function calling。  

![BV1aeLqzUE6L-[07:00]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-8.png)  


MCP实际上规范agent和tool的交互。  
![BV1aeLqzUE6L-[07:23]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-9.png)  
最开始将函数写在与agent同一个进程中。后来发现将工具函数放在server中，暴露api供各个客户端使用可以使得通用性大大增加。  
![BV1aeLqzUE6L-[08:08]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-11.png)  
MCP协议规定了client和server的通信格式和一些通用的接口，比如使用mcp就必须要让agent可以查询server有那些函数，这样的接口就由mcp这个协议来规定。 
![BV1aeLqzUE6L-[08:56]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-12.png)  
mcp server除了提供函数工具，还可用提供资源文件和prompts模板。  

llm使用mcp解决一个问题的全过程：  
1. 首先用户将prompt提供给agent  
![BV1aeLqzUE6L-[09:17]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-13.png)   

2. mcp server将所有工具的使用方法等信息编成自然语言格式的system prompt或者function calling的格式的system prompt。  
![BV1aeLqzUE6L-[09:20]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-15.png)
![BV1aeLqzUE6L-[09:22]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-14.png)
3. agent将system prompt和user prompt一起传给llm。  
![BV1aeLqzUE6L-[09:29]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-16.png)  
4. 如果是支持function calling的llm，则会按照function calling格式回复，那么agent可以很容易对llm的回复进行处理，调用相应的函数。  
![BV1aeLqzUE6L-[09:38]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-17.png)
5. 在这个例子中，agent调用了函数之后，得到的内容就是网页的内容，一般来说是html。这当然很容易作为文本再次发送给llm。  
![BV1aeLqzUE6L-[09:47]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-18.png)
6. llm在接收到相应的网页内容之后，就知道应该如何应对user的问题了。  
![BV1aeLqzUE6L-[10:03]](./images/f670df91-9ed9-4531-8d53-395087ce69f0-19.png)

这里我们可以试想以下：  
如果user提出的prompt是一个ai训练数据中没有的问题（比如当天某个地方发生了什么新闻）。  
没有mcp的情况下，llm大概率是回复“我无法提供。。。”  
而mcp的在llm端的本质就是提供给了更全面的prompt，除了user的问题，还提供了可能可以解决问题的方式，以及要求llm给出解决问题的步骤，这样一来llm大概率回复的就是使用mcp中的工具解决问题的步骤。而agent可以按照这些步骤去执行命令。而且这期间llm的所有回复都是展示出来的。    
那么设想一下什么时候停止这个过程呢？大概是agent检测到llm的输出中不再有调用函数的请求了。这样一来agent就不再由内容返回给llm，这个过程也就停止了。  
这样分析的话，agent本质上大概率是一套if-else的规则。  
而mcp就更加常规，本质上就是一系列的工具函数、数据、prompt模板。  






