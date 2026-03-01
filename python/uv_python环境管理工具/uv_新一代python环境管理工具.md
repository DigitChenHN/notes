## 初始化uv环境
![BV1JxLEzGEq5-[01:57]](./images/1959c736-807c-496e-9409-13a92f55d92b-2.png)

uv的思维：每一个项目文件夹是一个独立项目，需要建立一个新的环境。  
从uv创建环境的语句`uv init`来看，跟git比较类似。  
`uv init + [项目名称]（目前好像不支持中文）`会在当前文件夹下创建一个名为[项目名称]的文件夹，其目录结构：  
```
├── .gitignore
├── .python-version
├── README.md
├── main.py
└── pyproject.toml
```

1. 其创建的`.gitignore`文件会将python项目常见的一些不需要加入git版本管理的文件和文件夹排除在外，比如`__pycache__`文件夹、`.venv`文件夹、`.uv`文件夹等。  
![BV1JxLEzGEq5-[02:03]](./images/1959c736-807c-496e-9409-13a92f55d92b-3.png)  

2. `pyproject.toml`文件中记录了项目名称、项目描述、项目版本、当前项目python版本和各种依赖库的版本等信息。
![BV1JxLEzGEq5-[02:17]](./images/1959c736-807c-496e-9409-13a92f55d92b-4.png)  
3. `.python-version`文件则用来控制当前项目使用的python版本。这里的python版本将会控制uv为当前项目创建的虚拟环境的python版本。    
![BV1JxLEzGEq5-[02:09]](./images/1959c736-807c-496e-9409-13a92f55d92b-5.png)    

## 安装库  
`uv add package_name`命令会将一个名为package_name的包安装到当前项目的虚拟环境中，并且会将这个包的信息记录在pyproject.toml文件中。   
![BV1JxLEzGEq5-[02:45]](./images/1959c736-807c-496e-9409-13a92f55d92b-6.png)   
并且会创建一个uv.lock文件来记录安装这个包的过程的完整信息，有了这个文件就可以完整开发者的环境，从而顺利的运行代码。  

## uv具体管理环境的方式  
uv主要通过`.python-version`和`pyproject.toml`两个文件管理python。`.python-version`文件中记录了当前项目使用的python版本，`pyproject.toml`文件中记录了当前项目安装的库和版本要求。  
`uv.lock`文件类似于一个log，比如在环境中安装一个flask，flask会需要安装很多其他依赖，而安装各个依赖的过程会记录在`.lock`文件中。`.lock`文件中记录的信息比`pyproject.toml`文件中记录的信息更完整，`pyproject.toml`文件中只记录了安装flask这个包，而`.lock`文件中记录了安装flask这个包的过程中安装的所有依赖包及其精确版本信息（包括flask和flask的各种依赖）。  

## 一些常用的语句
1. `uv tree`  
![BV1JxLEzGEq5-[03:44]](./images/1959c736-807c-496e-9409-13a92f55d92b-7.png)
2. `uv remove`  
![BV1JxLEzGEq5-[04:02]](./images/1959c736-807c-496e-9409-13a92f55d92b-8.png)
3. `uv sync`  
手动修改`.python-version`和`pyproject.toml`文件之后，可以通过该命令应用这些修改。比如修改`.python-version`中的python版本，或者在`pyproject.toml`中添加某个依赖。  
4. `uv add -r requirements.txt`  
从一个名为requirements.txt的文件中安装依赖，并将其添加到`pyproject.toml`文件中。  
5. `uv lock --upgrade-packages package_name`
升级依赖的推荐方式，这个命令会尝试将指定的包更新到最新的兼容版本，同时能够保持`uv.lock`锁文件中的其他部分不变，尽量减少升级了依赖A之后发现依赖B又出问题的情况。


