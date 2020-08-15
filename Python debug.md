# Python debug
[TOC]

## Pycharm调试
### 本地调试
![](_v_images/20200801223457553_4661.png =807x)


#### Working directory
工作目录，即脚本是在哪个目录下被执行的，脚本中的**相对路径**均是基于这个工作目录计算的。

#### Add content roots to PYTHONPATH & Add source roots to PYTHONPATH
将 *content roots* 和 *source roots* 添加到 *PYTHONPATH* 环境变量中。

**何为 *PYTHONPATH***  
*PYTHONPATH* 环境变量中保存的是一组路径，Python 解释器会从这些路径中搜索我们 import 的模块。在我们的 Python 脚本中可以通过 `sys.path` （这是一个 `list(str)`）去查看和修改。  
希望 Python 能够找到我们自己的模块，一般有三种方法：
- 将我们模块的路径添加到 *PYTHONPATH* 环境变量中
- 在 import 我们的模块之前，先执行语句：`sys.path.append('/path/to/our/module')`
- 在 Python 的以下四个路径之一里添加一个 `.pth` 文件，这个文件中每一行代表一个模块在系统中的绝对路径：  
    - `sys.prefix`：和机器平台无关的 Python 文件的安装地址
    - `sys.exec_prefix`：和机器平台相关的 Python 文件的安装地址，默认情况下和上面相同
    - `prefix/lib/site-packages`（Windows平台）：prefix 代表 Python 的安装路径，这个目录中主要是我们手动安装的模块，如使用命令 `python setup.py install` 或者使用 `pip` 安装的模块。
    - `prefix/lib/pythonX.Y/site-packages`（Unix平台）：同上。


**何为 *content root***  
在 Jetbrains 全家桶中 *content* 代表你正在使用的文件（项目文件）的集合，这些文件通常是以不同的文件夹层级来组织的，最上层的文件夹则被称为 *content root* 。通常使用 ![content root](https://www.jetbrains.com/help/img/idea/2020.2/icons.nodes.folder.png) 图标表示。在一个项目中默认至少有一个 *content root* 文件夹，即项目的根目录。  
当然，一个项目中也可以有多个 *content root* ，例如我们可以指定一个存储所有 icon 文件的目录作为项目额外的 *content root* 。  

*content root* 中的文件会被建立索引，这些文件可以很方便地被我们搜索，IDE 也会去尝试解析这些文件，建立这些文件和模块之间的联系，并将这些文件的信息用于代码补全功能。*content root* 又可以细分为下面几类：

- *Regular content root*   ![content root](https://www.jetbrains.com/help/img/idea/2020.2/icons.nodes.folder.png)：包含组成项目的文件，项目的根目录就是一个 *Regular content root* 。   
- *Source root* ![Source root](https://www.jetbrains.com/help/img/idea/2020.2/icons.modules.sourceRoot.png)：主要包含源代码文件，Pycharm中将 *source root* 作为解析 **import** 的起始目录。**当我们项目中明明有这个模块，但是IDE却提示找不到类或者变量，可以检查下对应模块的源码文件是不是位于 *source root*** 中。
- *Resource root* ![Resource roots](https://www.jetbrains.com/help/img/idea/2020.2/rootResourceIJ.png) ![Resource roots](https://www.jetbrains.com/help/img/idea/2020.2/icons.modules.resourcesRoot.png)：包含项目中的资源文件（图像、样式表、配置文件等），如果一个文件夹被指定为 *resource root* ，引用该文件夹下的文件或者目录时不需要指定完整的路径，仅需要指定相对于这个文件夹的**相对路径**即可。
- *Excluded root* ![Excluded root](https://www.jetbrains.com/help/img/idea/2020.2/icons.modules.excludeRoot.png)：忽略这些目录中的文件，不建立索引，不搜索，不解析。一般可以 exclude 临时的 build 文件夹，生成的输出，日志等，这样可以极大提高 IDE 的性能表现。
- *Template roots* ![Template root](https://www.jetbrains.com/help/img/idea/2020.2/template_folder_icon.png)：里面存放的主要是 web 项目的模板文件。

#### Redirect input from
这个配置主要是将 Python 程序的标准输入重定向到某个文件，当 Python 程序试图从标准输入流读取数据时，读取的就是这个文件中的数据。  
**这里需要注意的是，当达到文件末尾时，Python 程序会读取到一个 `EOF`。当读取到 `EOF` 时，无论是 Python 2 中的 `raw_input()` 还是 Python 3 中的 `input()`，这两个方法的行为都是抛出一个 `EOFError` 异常，可能会打断正常的程序流程。**  

### 远程调试
当我们的 Pyhton 脚本需要运行在服务器中，无法在本地进行调试时，我们可以利用 Pycharm 自带的远程调试功能。  
![](_v_images/20200802001132951_14589.png =807x)

Pycharm 远程调试的原理是在 Pycharm 中开启一个 debug 服务器，当 Python 脚本运行时主动与我们的 debug 服务器连接，接收我们设置断点、堆栈查看、变量修改等命令。这种 debug 方式有一定的侵入性，我们需要修改 Python 脚本，使其在运行起来后主动连接 debug 服务器。  

1. 首先需要将 Pycharm 安装路径下的 `pycharm-debug.egg` 文件上传到 Python 运行环境中，并将其添加到 *PYTHONPATH* 变量中。  
2. 然后在脚本中添加如下语句，使其能够主动连接 debug 服务器。  
```python
import pydevd
pydevd.settrace('192.168.2.1', port=2333, stdoutToServer=True, stderrToServer=True)
```
方法中的前两个参数为 debug 服务器（本地）的 IP 地址和端口号，这里需要先手动安装一下 *pydevd* ：`pip install pydevd`。  
3. 准备就绪后，先点击 Pycharm 中的 Debug 按钮启动 debug 服务器。  
![](_v_images/20200802002048261_6366.png =600x)  
4. 最后运行远程主机上的 Python 程序，其在运行之后会主动连接 debug 服务器，并会暂停在 `pydevd.settrace()` 方法之后的第一条语句上。  
![](_v_images/20200802005418271_25109.png =641x)  

**注意**：在第一次远程调试时，会出现如下的提示信息：  
![](_v_images/20200802003044174_20516.png =600x)  
这是因为远程主机中的 Python 脚本的路径与我们本地的路径不同导致的，需要映射一下，上图中提供了三种解决办法：

- Edit settings：在 debug 设置中手动指定路径的映射，一般只需要映射两台主机上项目的根目录即可。  
![](_v_images/20200802003511285_27052.png =744x)
- Auto-detect：让 Pycharm 自己去探测可能的映射。  
![](_v_images/20200802003646233_28536.png =436x)
- Download：从远程机器上下载源代码，这样 Pycharm 就可以知道对应关系了。  


## PDB
PDB 是 Python 中自带的一个模块，是一个交互式的 Python 调试器，支持设置断点、单步执行、栈帧检查和源码查看，并且支持在任意的栈帧中执行任意的 Python 代码。它也支持调试已经挂掉的 Python 程序（验尸模式），并且也可以在程序内部调用。  

### 进入调试
可以通过以下命令来调试一个 Python 程序：  
```shell
python3 -m pdb myScript.py
```

当 pdb 被像上面这样作为一个脚本来调用时，它会在 `myScript.py` 异常退出时主动进入验尸模式。当我们结束验尸模式或者程序正常退出时，pdb 会自动重新启动被调试的脚本，并且之前的断点和状态都会被保留。  

当然我们也可以在我们的程序中加入一行代码来主动进入 pdb 的 debug 模式：  
```shell
import pdb: pdb.set_trace()
```
这行代码可以加入到我们想打断点的地方。  

pdb 模块中定义了一系列可以使 Python 程序进入 debug 模式的函数：  

- `pdb.run(statement, globals=None, locals=None)`  
在 debugger 中运行 *statement* （一个 string 或者一个 code 对象），*globals* 和 *locals* 是程序运行的环境，  

- `pdb.runeval(expression, globals=None, locals=None)`  
在 debugger 计算一个 *expression* （一个 string 或者一个 code 对象）。  

- `pdb.runcall(function, *args, **kwds)`  
使用给定参数调用一个 *function* ，会在进入 *function* 时进入 debugger。  

- pdb.set_trace(*, header=None)  
直接在当前栈帧进入 debugger，这个是硬编码设置断点的方式，即使当前是正常运行 Python 脚本也会触发进入 debugger。如果设置了`header` 则会在进入 debugger 之后打印出来。  
- `pdb.post_mortem(traceback=None)`  
根据给定的 *traceback* 对象进入验尸模式，如果没有给定 *traceback* 对象，则会根据当前正在处理的异常进入。进入验尸模式后会回到抛出异常的那一个栈帧。  
- `pdb.pm()`  
利用存储在 `sys.last_traceback` 中的 *traceback* 进入验尸模式。  

### 调试命令
大部分的命令都有简写的形式，命令与参数之间通过空格或者 tab 分隔，可选参数会用 `[]` 包裹，互斥的参数则使用 `|` 表示。  

在前面已经执行过命令之后直接回车会重复上一条命令。  

对于不属于 pdb 的命令则会被当成 Python 表达式在当前的上下文环境中执行，可以改变当前栈帧中的变量，也可以调用一个函数。也可以在表达式之前加上 `!` 显式声明这是一个 Python 表达式。当执行过程中触发异常时，只会将异常打印出来，不会改变 pdb 的状态。  

- `h(elp) [command]`  
跟 Python 中的 `help()` 函数很像，打印出给定指令的帮助文档，如何没有指定指令，则打印出所有可用的指令。  
- `w(here)`  
打印出堆栈信息，最近的栈帧打印在最下面，当前栈帧会用箭头标识出来，栈帧中的内容决定了大部分命令的上下文。  
- `d(own) [count]`  
往下移动 `count` 栈帧，即移动到更新的栈帧中。  
- `u(p) [count]`  
往上移动 `count` 栈帧，即移动到更老的栈帧中。  
- `b(reak) [([filename:]lineno | function) [, condition]]`  
添加断点，`filename` 代表文件名，默认当前运行的文件，`lineno` 为行号，也可以直接指定一个 `function` 断点位置即为函数的入口处。`condition` 为断点的条件，是一个表达式。如果命令没有任何参数，则会列出所有的断点信息。每个断点都会对应一个数字，可以用来快速引用。  
- `tbreak [([filename:]lineno | function) [, condition]]`  
临时断点，用法和上面那个一样，只是会在第一次命中之后移除。  
- `cl(ear) [filename:lineno | bpnumber [bpnumber ...]]`  
当使用 *filename:lineno* 作为参数时，会清除掉这一行的所有断点，当使用 *bpnumber* 作为参数时会清除掉所有指定的断点，如果没有参数则是清除掉所有断点，会事先确认。  
- `disable [bpnumber [bpnumber ...]]`  
禁用给定的断点，但是不会清除，之后可以再次启用。  
- `enable [bpnumber [bpnumber ...]]`  
启用给定的断点。  
- `ignore bpnumber [count]`  
忽略给定断点的前 *count* 次命中，每命中一次断点 *count* 就会减 1，当减为 0 后断点就会启用。  
- `condition bpnumber [condition]`  
为断点设置一个条件，会覆盖原来的条件。如果没有指定 *condition* 则会去掉断点的条件。  
- `commands [bpnumber]`  
