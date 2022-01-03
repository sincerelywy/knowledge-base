# TextFSM

> 做事要明确目的，已经过了广撒网的时代了。

原意是向把运维的网络设备（交换机、路由器、负载均衡器）的ARP、MAC、路由等等信息保存到一个Git仓库里，这样回查方便。于是发现了Google开源的TextFSM，感觉是一个不错的工具，于是尝试。

当然，作为一名（曾经）专业的码农，既然是Python的库，学习方式自然是直奔官网。

## TextFSM是什么

TextFSM就是Google用来处理通过CLI获取的网络设备信息。有个词比较关键：`semi-formatted text`，半格式化的文本，这是使用TextFSM的输入内容。当然TextFSM模板文件也是输入。两者结合最终输出想要的数据，并且格式化。

## TextFSM用法

直接上示例：

```python
# Run text through the FSM. 
# The argument 'template' is a file handle and 'raw_text_data' is a string.
re_table = textfsm.TextFSM(template)
data = re_table.ParseText(raw_text_data)

# Display result as CSV
# First the column headers
print( ', '.join(re_table.header) )
# Each row of the table.
for row in data:
  print( ', '.join(row) )
```

TextFSM这个库也能直接用看检查模板和输入是否能够得到正确输出，也就是检查是否符合预期：`parser.py [--help] template [input_file [output_file]]`，文件一般在`dist-packages/textfsm/parser.py`。

## 数据

FSM是用来提取处文本数据（textual data）中想要的数据（essential data），然后输出成表格（tabular）。个人简单理解类似格式化，洗数据，ETL中的Transform。原文这么说的：

> At its core, the FSM's purpose is to extract the essential data from a textual data and place it into a tabular representation.

比如下面的数据：

```markdown
Routing Engine status:
  Slot 0:
    Current state                  Master
    Election priority              Master (default)
    Temperature                 39 degrees C / 102 degrees F
    CPU temperature             55 degrees C / 131 degrees F
    DRAM                      2048 MB
    Memory utilization          76 percent
    CPU utilization:
      User                      95 percent
      Background                 0 percent
      Kernel                     4 percent
      Interrupt                  1 percent
      Idle                       0 percent
    Model                          RE-4.0
    Serial ID                      xxxxxxxxxxxx
    Start time                     2008-04-10 20:32:25 PDT
    Uptime                         180 days, 22 hours, 45 minutes, 20 seconds
    Load averages:                 1 minute   5 minute  15 minute
                                       0.96       1.03       1.03
Routing Engine status:
  Slot 1:
    Current state                  Backup
    Election priority              Backup
    Temperature                 30 degrees C / 86 degrees F
    CPU temperature             31 degrees C / 87 degrees F
    DRAM                      2048 MB
    Memory utilization          14 percent
    CPU utilization:
      User                       0 percent
      Background                 0 percent
      Kernel                     0 percent
      Interrupt                  1 percent
      Idle                      99 percent
    Model                          RE-4.0
    Serial ID                      xxxxxxxxxxxx
    Start time                     2008-01-22 07:32:10 PST
    Uptime                         260 days, 10 hours, 45 minutes, 39 seconds
```

如果想要card state和temperature，制定好FSM的处理模板，FSM处理后的结果返回成tabular，各个数据之间能够作为一个记录放在一起：

|Slot|Model|Dram|State|Temp|CPUTemp|
|---|---|---|---|---|---|
|0|RE-4.0|2048|Master|39|55|
|1|RE-4.0|2048|Backup|30|31|

## TextFSM模板文件

官方给的示例模板文件如下：

```markdown
# Chassis value will be null for single chassis routers.
Value Filldown Chassis (.cc.?-re.)
Value Required Slot (\d+)
Value State (\w+)
Value Temp (\d+)
Value CPUTemp (\d+)
Value DRAM (\d+)
Value Model (\S+)

# Allway starts in 'Start' state.
Start
  ^${Chassis}
  # Record current values and change state.
  # No record will be output on first pass as 'Slot' is 'Required' but empty.
  ^Routing Engine status: -> Record RESlot

# A state transition was not strictly necessary but helpful for the example.
RESlot
  ^\s+Slot\s+${Slot}
  ^\s+Current state\s+${State}
  ^\s+Temperature\s+${Temp} degrees
  ^\s+CPU temperature\s+${CPUTemp} degrees
  ^\s+DRAM\s+${DRAM} MB
  # Transition back to Start state.
  ^\s+Model\s+${Model} -> Start

# An implicit EOF state outputs the last record.
```

模板文件有两部分组成：

- Value，描述了要提取的内容。
- 至少一个的"State"定义，`describing the various states of the engine whilst parsing data.`，在处理数据的同时描述了engine的各种状态。*这里的engine目前我暂时理解成TextFSM的处理程序。*

任意空格开始然后跟着`#`，也即匹配`^\s*#`的都会被认为是注释行。

### Value

Value必须在State前面描述，而且必须是连续的行，中间只允许注释。格式：

`Value [option[,option...]] name regex`

- Value，Keyword，表明这是一个Value行，强制性（mandatory）。
- option，Flags，逗号分隔comma separated (no spaces)，关于Value的可选项，可能是下面的一个或者多个：
  - Filldown。前面被匹配的Value会被保留到后面的记录，除非被明确清除或者再次被匹配。也就是说最近被匹配到的Value会被复制到新的行，除非再次被匹配。
  - Key。
  - Required。只有当Value被匹配了，记录（row）才会被保存到表中。
  - List。表明Value是列表，每个被匹配的都会被append。通常情况Value只会记录最新被匹配的，也就是说会覆盖之前在当前行被匹配的结果。
  - Fillup。和Filldown类似，只不过是向上填充，直到找到非空的条目。和Required不兼容。
- name，Value的名称，同时也是最后结果表的列名。注意不要和option的关键字冲突了。
- regex，表明Value要匹配的正则表达式，必须在括号内。

### State

每段State定义都会用空行分隔，起始行是一个数字字母组成的State name，后面跟着一系列的rule，注意每个rule行开头都需要有一个或者两个空格，外加`^`：

```yml
stateName
 ^rule
 ^rule
 ...
```

FSM从Start state开始，输入的内容从这个State开始，会逐行匹配，直到遇到EOF或者当前State到达End。匹配到的行可以触发转换到新的State。

### Reserved states

按照上面的描述，Start state必须有。

如果输入到达EOF，那么EOF state会被触发，这是个隐含的状态，效果类似：

```yml
EOF
  ^.* -> Record
```

EOF会在返回前记录当前行。也可以显式声明一个EOF state来覆盖覆盖这一个特性：

```yml
EOF
```

END state是默认的，用来终止输入行，并且不执行EOF state。

### State Rules

FSM从输入中读取一行，从当前State的起始规则，顺序对每条rule进行测试。如果输入匹配到了一个rule，那么执行action，然后再从输入中读取一行，重新从当前State起始规则开始匹配，也就是不断循环。

rule的格式：`^regex [-> action]`

regex就是正则表达式。包含了任意个Value描述符（可以是0），使用`${ValueName}`（官方推荐这种。当然也可以`$ValueName`）格式。把Value对应的正则带入到rule里，如果输入行匹配了rule，则将与此值匹配的文本作为给当前行的匹配结果。如果要显式行尾EOL，使用`$$`，这样会在Value带入的时候替换成单个$。

比如下面的模板：

```yml
Value Interface (\S+)

Start
  ^Interface ${Interface} is up
```

在开始解析FSM的时候，FSM会把rule扩展成：`^Interface (\S+) is up`。那么当`Interface GigabitEthernet1/10 is up.`按照这个rule解析的时候，Interface会被赋值"GigabitEthernet1/10"。感觉和正则表达式的捕获组是不是挺像的。

注意，可以在一个rule里使用多个Value，但是rule必须被整个匹配，其中的Value才能被赋值。要想Value有值，整个正则表达式得能够匹配。

### Rule Actions

正则表达式后面跟的是action，使用`->`，格式`A.B C`。如果没有指定action，那么使用默认的Next.NoRecord。

action分为三个可选部分：

- Line。输入行的action，也就是对应格式里的`A`部分。
  - Next，完成输入行，读入下一行并从状态开始再次开始匹配。默认的Line action，不指定就用这个。
  - Continue，保留当前行，不从状态的第一条规则恢复匹配。继续处理规则，就好像没有发生匹配一样（仍然发生值分配）。也就是说让当前行往下继续执行。
- Record。Line action后，可选的action，英文句号分割，也就是对应格式里的`B`部分。
  - NoRecord。啥都不做。这个也是默认的Record action。
  - Record。将到目前为止匹配的值为返回数据中的一行。清除Filldown之外的Value。注意：如果有Required value没被赋值，那么就没有任何输出。也就是说Required value必须被匹配否则就没有结果。
  - Clear。清除Filldown之外的Value。
  - Clearall。清除所有Value。
- New State Transition。也就是对应格式里的`C`部分。空格后是新的State。必须是模板定义的Reserved state或者有效的State。在任意action执行后，从输入中读取下一行，然后将当前State更改为新State，并在此新State下继续处理。注意：State Transition不兼容Continue action。

如果Line和Record都存在，需要使用`.`符号。如果Line或者Record action使用了默认值，那么`.`符号可以省略。例如：Next、Next.NoRecord、NoRecord这三个是相同的。

除此之外，还有一个Error Action，简单理解就是报异常，结束所有操作，不输出任何结果。格式：`^regex -> Error [word|"string"]`。

## 参考

- [TextFSM](https://github.com/google/textfsm)：官方Repo。
- [网工手艺](https://zhuanlan.zhihu.com/p/370526806)：一个网工自动化成长之路。
