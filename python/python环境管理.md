# 引言
个人理解，解释型语言在运行是逐行将代码翻译成机器语言然后运行，而解释器往往有很多版本，人们根据不同的解释器作为依赖来构建代码，而其中某些代码又可以作为依赖被其他人在项目中调用来构建自己的项目代码。因为是解释型语言，所以可以不需要编译打包，而是直接按照项目开发者要求的逐一安装好对应版本的所有依赖，就可以直接运行项目了。而不同的项目可能需要使用到不同的版本的依赖，在同一台机器上安装多个版本的依赖会产生冲突，就需要使用虚拟环境来隔离项目所需要的依赖。    
> 而编译型语言在运行前需要使用特定的编译器将代码编译成机器码，编译成机器码之后运行就不需要依赖编译器，但是机器码也并非通用的，不同的CPU架构的机器码是不同的。  

# venv
由于python语法相对简单，开源社区非常活跃，发展速度很快，各种库不断涌向，所以非常需要环境管理工具来管理这些库的版本和依赖关系。在python3.3之后开始内置了venv环境管理工具，venv的使用很简单，使用`python -m venv env_name`命令就可以创建一个名为env_name的虚拟环境了。  
> 创建虚拟环境的原理很简单，就是venv会在当前目录下创建一个env_name文件夹，里面包含了python解释器和一些必要的库文件。而运行项目时，指定其中的python作为解释器，这样的话就会优先使用这个env_name文件夹中的python解释器和库文件，从而与当前操作系统中存在的其他python解释器和库文件隔离开来。  
> venv用来创建虚拟环境的文件夹中，主要包含了bin文件夹（Windows系统中是Scripts文件夹）和Lib文件夹。bin文件夹中包含了python解释器、pip以及激活虚拟环境的脚本；Lib文件夹中包含了python的标准库。Lib中有site-packages文件夹，里面包含了安装的第三方库。直接删除这个文件夹就删除了这个虚拟环境。  
由于venv是当前python版本的内置模块，所以只能创建和当前版本一致的虚拟环境。  

# conda 
大多数人知道conda可能是通过Anaconda。Anaconda是一个开源的python发行版（可以理解为一个定制化的python环境），由于是专门为数据科学领域和机器学习领域设计的，所以内置了很多数据科学和机器学习领域常用的库，其使用conda来管理各种库和环境。  
conda是用python语言开发的跨平台、语言无关的包管理和环境管理工具，既可以管理python的环境和库，也可以管理其他语言比如R语言的环境和库。  
对于用户端，需要了解的conda管理包和虚拟环境的内容比较简单：  
1. `conda create -n env_name`：创建一个名为env_name的虚拟环境。  
2. `conda activate env_name`：激活一个名为env_name的虚拟环境。  
3. `conda install package_name`：将一个名为package_name的包安装到当前激活的虚拟环境中。    


在用户的机器上管理环境和包的本质就是在conda的安装目录下的envs文件夹中创建一个名为env_name的文件夹，其项目结构和python安装目录类似，包含了python解释器、Lib文件夹、libs文件夹、include文件夹等，相比于venv创建的目录更加地完整，因为是一个独立版本的python。  
使用`conda activate env_name`命令激活一个虚拟环境的原理就是修改当前shell的环境变量（Windows的cmd、powershell，Linux和macOS的bash或zsh都叫shell），将当前shell的PATH环境变量指向env_name文件夹中的python解释器和库文件，从而优先使用这个env_name文件夹中的python解释器和库文件。  
conda的缺点是安装包比较大，安装和更新包的速度比较慢，而且有时候会出现依赖冲突的问题。  

# uv
uv是python3.11之后新推出的一个环境管理工具，与conda主要关注于数据科学领域不同，uv主要关注于python开发领域。相比于conda的虚拟环境是放在conda安装目录下的，uv的设计理念是每一个项目文件夹都是一个独立的项目，所以其虚拟环境是放在各个项目文件夹下的。因此其创建虚拟环境的方式是在当前目录下创建一个`.venv`文件夹，这个文件夹和venv创建的类似，主要也是由Scripts文件夹和Lib文件夹组成。  
> 关于uv的具体使用可以参考[uv--环境管理工具](/python/uv_python环境管理工具/uv_新一代python环境管理工具.md)。  

# 依赖管理
创建虚拟环境方便了项目在某一台机器上运行，当时如果要让项目能够被更多人运行，就需要将项目依赖管理好，让其他人能够根据信息重现能够运行项目的环境。  
## venv
包管理工具pip和venv一样是python内置的工具，主要用于安装和管理python包，可以用于从指定源（比如pypi）下载和安装python包。激活虚拟环境之后，使用`pip install`可以向当前shell环境中的python环境安装python包。
常用的命令还有:   
`pip list`：列出当前环境中安装的所有包和版本信息。  
`pip freeze > requirements.txt`：将当前环境中的包和版本信息导出到一个名为requirements.txt的文件中。  
`pip install -r requirements.txt`：从一个名为requirements.txt的文件中安装。  

一个requirements.txt文件一般是这样（既可以直接指定版本，也可以指定版本范围，还可以不指定版本）：    
```
tensorflow==2.12.0
numpy==2.0.0
pandas>=2.0.0
scikit-learn
```
requirements.txt文件往往用来定义项目的核心依赖，往往不会限定某个包的版本，同时也一般不会去记录依赖的依赖，尤其是开发者在手动写这个requirements.txt文件的时候。  
这时候往往会使用一个`constraints.txt`来限定和控制依赖的版本（包括某些深层依赖）。`pip install -c constraints.txt -r requirements.txt`会先安装requirements.txt中指定的包，当安装其中的某个依赖A的依赖B时，如果B在constraints.txt中有指定版本要求，那么就会安装constraints.txt中指定的版本。而安装过程中没有涉及到的依赖，即便在constraints.txt中有指定版本要求，也不会安装。   
## conda
使用conda激活虚拟环境之后，同样可以使用pip安装和管理包。不过conda本身也具备安装和管理包的功能。  
相关的常用命令有：  
1. `conda list`：列出当前虚拟环境中安装的所有包和版本信息。  
2. `conda remove package_name`：将一个名为package_name的包从当前虚拟环境中卸载。  
3. `conda env export > environment.yml`：将当前虚拟环境中的包和版本信息导出到一个名为environment.yml的文件中。
4. `conda info --envs`：列出当前机器上所有的虚拟环境。  
5. `conda env remove -n env_name`：删除一个名为env_name的虚拟环境。  

## uv 
uv在项目文件夹中创建`pyproject.toml`、`.python-version`和`uv.lock`三个文件来管理项目的依赖和环境。其中`pyproject.toml`文件中记录了项目名称、项目描述、项目版本、当前项目python版本和各种依赖库的版本等信息；`.python-version`文件则用来控制当前项目使用的python版本。`uv.lock`文件则记录安装依赖的详细过程，包括依赖的名称、版本、源、依赖关系等信息。   
`uv.lock`文件对于解决依赖冲突非常有用。  
使用`uv lock --upgrade-packages package_name`这种方式是nv推荐的升级依赖的方式，这个命令会尝试将指定的包更新到最新的兼容版本，同时能够保持`uv lock`锁文件中的其他部分不变，尽量减少升级了依赖A之后发现依赖B又出问题的情况。  
其他常用的一些命令：  
1. `uv add package_name`：将一个名为package_name的包安装到当前项目的虚拟环境中，并将其添加到`pyproject.toml`文件中。
2. `uv init`：在当前目录下初始化几个主要的文件，包括`.gitignore`、`.python-version`和`pyproject.toml`文件。
3. `uv init + [项目名称]`：在当前目录下创建一个名为[项目名称]的文件夹，并在其中初始化几个主要的文件，包括`.gitignore`、`.python-version`和`pyproject.toml`文件。
4. `uv add -r requirements.txt`：从一个名为requirements.txt的文件中安装依赖，并将其添加到`pyproject.toml`文件中。 
   