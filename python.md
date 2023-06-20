# ⌚ python

## 1、常用模块

### os

os模块负责程序与操作系统的交互，提供了访问操作系统底层的接口。

```python
os.remove() 删除文件 
os.unlink() 删除文件 
os.rename() 重命名文件 
os.listdir() 列出指定目录下所有文件 
os.chdir() 改变当前工作目录
os.getcwd() 获取当前文件路径
os.mkdir() 新建目录
os.rmdir() 删除空目录(删除非空目录, 使用shutil.rmtree())
os.makedirs() 创建多级目录
os.removedirs() 删除多级目录
os.stat(file) 获取文件属性
os.chmod(file) 修改文件权限
os.utime(file) 修改文件时间戳
os.name(file) 获取操作系统标识
os.system() 执行操作系统命令
os.execvp() 启动一个新进程
os.fork() 获取父进程ID，在子进程返回中返回0
os.execvp() 执行外部程序脚本（Uinx）
os.spawn() 执行外部程序脚本（Windows）
os.access(path, mode) 判断文件权限(详细参考cnblogs)
os.wait() 暂时未知
os.path模块：
os.path.split(filename) 将文件路径和文件名分割(会将最后一个目录作为文件名而分离)
os.path.splitext(filename) 将文件路径和文件扩展名分割成一个元组
os.path.dirname(filename) 返回文件路径的目录部分
os.path.basename(filename) 返回文件路径的文件名部分
os.path.join(dirname,basename) 将文件路径和文件名凑成完整文件路径
os.path.abspath(name) 获得绝对路径
os.path.splitunc(path) 把路径分割为挂载点和文件名
os.path.normpath(path) 规范path字符串形式
os.path.exists() 判断文件或目录是否存在
os.path.isabs() 如果path是绝对路径，返回True
os.path.realpath(path) #返回path的真实路径
os.path.relpath(path[, start]) #从start开始计算相对路径 
os.path.normcase(path) #转换path的大小写和斜杠
os.path.isdir() 判断name是不是一个目录，name不是目录就返回false
os.path.isfile() 判断name是不是一个文件，不存在返回false
os.path.islink() 判断文件是否连接文件,返回boolean
os.path.ismount() 指定路径是否存在且为一个挂载点，返回boolean
os.path.samefile() 是否相同路径的文件，返回boolean
os.path.getatime() 返回最近访问时间 浮点型
os.path.getmtime() 返回上一次修改时间 浮点型
os.path.getctime() 返回文件创建时间 浮点型
os.path.getsize() 返回文件大小 字节单位
os.path.commonprefix(list) #返回list(多个路径)中，所有path共有的最长的路径
os.path.lexists #路径存在则返回True,路径损坏也返回True
os.path.expanduser(path) #把path中包含的”~”和”~user”转换成用户目录
os.path.expandvars(path) #根据环境变量的值替换path中包含的”$name”和”${name}”
os.path.sameopenfile(fp1, fp2) #判断fp1和fp2是否指向同一文件
os.path.samestat(stat1, stat2) #判断stat tuple stat1和stat2是否指向同一个文件
os.path.splitdrive(path) #一般用在windows下，返回驱动器名和路径组成的元组
os.path.walk(path, visit, arg) #遍历path，给每个path执行一个函数详细见手册
os.path.supports_unicode_filenames() 设置是否支持unicode路径名
```

### sys

sys模块包括了一组非常实用的服务，内含很多函数方法和变量，用来处理Python运行时配置以及资源，从而可以与前当程序之外的系统环境交互，如：python解释器。

```python
sys.argv 命令行参数List，第一个元素是程序本身路径 
sys.path 返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值 
sys.modules.keys() 返回所有已经导入的模块列表
sys.modules 返回系统导入的模块字段，key是模块名，value是模块 
sys.exc_info() 获取当前正在处理的异常类,exc_type、exc_value、exc_traceback当前处理的异常详细信息
sys.exit(n) 退出程序，正常退出时exit(0)
sys.hexversion 获取Python解释程序的版本值，16进制格式如：0x020403F0
sys.version 获取Python解释程序的版本信息
sys.platform 返回操作系统平台名称
sys.stdout 标准输出
sys.stdout.write(‘aaa‘) 标准输出内容
sys.stdout.writelines() 无换行输出
sys.stdin 标准输入
sys.stdin.read() 输入一行
sys.stderr 错误输出
sys.exc_clear() 用来清除当前线程所出现的当前的或最近的错误信息 
sys.exec_prefix 返回平台独立的python文件安装的位置 
sys.byteorder 本地字节规则的指示器，big-endian平台的值是‘big‘,little-endian平台的值是‘little‘ 
sys.copyright 记录python版权相关的东西 
sys.api_version 解释器的C的API版本 
sys.version_info ‘final‘表示最终,也有‘candidate‘表示候选，表示版本级别，是否有后继的发行 
sys.getdefaultencoding() 返回当前你所用的默认的字符编码格式 
sys.getfilesystemencoding() 返回将Unicode文件名转换成系统文件名的编码的名字 
sys.builtin_module_names Python解释器导入的内建模块列表 
sys.executable Python解释程序路径 
sys.getwindowsversion() 获取Windows的版本 
sys.stdin.readline() 从标准输入读一行，sys.stdout.write(“a”) 屏幕输出a
sys.setdefaultencoding(name) 用来设置当前默认的字符编码(详细使用参考文档) 
sys.displayhook(value) 如果value非空，这个函数会把他输出到sys.stdout(详细使用参考文档)
```

### re

```python
一.常用正则表达式符号和语法：
'.' 匹配所有字符串，除\n以外
‘-’ 表示范围[0-9]
'*' 匹配前面的子表达式零次或多次。要匹配 * 字符，请使用 \*。
'+' 匹配前面的子表达式一次或多次。要匹配 + 字符，请使用 \+
'^' 匹配字符串开头
‘$’ 匹配字符串结尾 re
'\' 转义字符， 使后一个字符改变原来的意思，如果字符串中有字符*需要匹配，可以\*或者字符集[*] re.findall(r'3\*','3*ds')结['3*']
'*' 匹配前面的字符0次或多次 re.findall("ab*","cabc3abcbbac")结果：['ab', 'ab', 'a']
‘?’ 匹配前一个字符串0次或1次 re.findall('ab?','abcabcabcadf')结果['ab', 'ab', 'ab', 'a']
'{m}' 匹配前一个字符m次 re.findall('cb{1}','bchbchcbfbcbb')结果['cb', 'cb']
'{n,m}' 匹配前一个字符n到m次 re.findall('cb{2,3}','bchbchcbfbcbb')结果['cbb']
'\d' 匹配数字，等于[0-9] re.findall('\d','电话:10086')结果['1', '0', '0', '8', '6']
'\D' 匹配非数字，等于[^0-9] re.findall('\D','电话:10086')结果['电', '话', ':']
'\w' 匹配字母和数字，等于[A-Za-z0-9] re.findall('\w','alex123,./;;;')结果['a', 'l', 'e', 'x', '1', '2', '3']
'\W' 匹配非英文字母和数字,等于[^A-Za-z0-9] re.findall('\W','alex123,./;;;')结果[',', '.', '/', ';', ';', ';']
'\s' 匹配空白字符 re.findall('\s','3*ds \t\n')结果[' ', '\t', '\n']
'\S' 匹配非空白字符 re.findall('\s','3*ds \t\n')结果['3', '*', 'd', 's']
'\A' 匹配字符串开头
'\Z' 匹配字符串结尾
'\b' 匹配单词的词首和词尾，单词被定义为一个字母数字序列，因此词尾是用空白符或非字母数字符来表示的
'\B' 与\b相反，只在当前位置不在单词边界时匹配
'(?P<name>...)' 分组，除了原有编号外在指定一个额外的别名 re.search("(?P<province>[0-9]{4})(?P<city>[0-9]{2})(?P<birthday>[0-9]{8})","371481199306143242").groupdict("city") 结果{'province': '3714', 'city': '81', 'birthday': '19930614'}
[] 是定义匹配的字符范围。比如 [a-zA-Z0-9] 表示相应位置的字符要匹配英文字符和数字。[\s*]表示空格或者*号。
二.常用的re函数：
方法/属性 作用
re.match(pattern, string, flags=0) 从字符串的起始位置匹配，如果起始位置匹配不成功的话，match()就返回none
re.search(pattern, string, flags=0) 扫描整个字符串并返回第一个成功的匹配
re.findall(pattern, string, flags=0) 找到RE匹配的所有字符串，并把他们作为一个列表返回
re.finditer(pattern, string, flags=0) 找到RE匹配的所有字符串，并把他们作为一个迭代器返回
re.sub(pattern, repl, string, count=0, flags=0) 替换匹配到的字符串
```

### datetime

```python
datetime.date.today() 本地日期对象,(用str函数可得到它的字面表示(2014-03-24))
datetime.date.isoformat(obj) 当前[年-月-日]字符串表示(2014-03-24)
datetime.date.fromtimestamp() 返回一个日期对象，参数是时间戳,返回 [年-月-日]
datetime.date.weekday(obj) 返回一个日期对象的星期数,周一是0
datetime.date.isoweekday(obj) 返回一个日期对象的星期数,周一是1
datetime.date.isocalendar(obj) 把日期对象返回一个带有年月日的元组
datetime对象：
datetime.datetime.today() 返回一个包含本地时间(含微秒数)的datetime对象 2014-03-24 23:31:50.419000
datetime.datetime.now([tz]) 返回指定时区的datetime对象 2014-03-24 23:31:50.419000
datetime.datetime.utcnow() 返回一个零时区的datetime对象
datetime.fromtimestamp(timestamp[,tz]) 按时间戳返回一个datetime对象，可指定时区,可用于strftime转换为日期表示 
datetime.utcfromtimestamp(timestamp) 按时间戳返回一个UTC-datetime对象
datetime.datetime.strptime(‘2014-03-16 12:21:21‘,”%Y-%m-%d %H:%M:%S”) 将字符串转为datetime对象
datetime.datetime.strftime(datetime.datetime.now(), ‘%Y%m%d %H%M%S‘) 将datetime对象转换为str表示形式
datetime.date.today().timetuple() 转换为时间戳datetime元组对象，可用于转换时间戳
datetime.datetime.now().timetuple()
time.mktime(timetupleobj) 将datetime元组对象转为时间戳
time.time() 当前时间戳
time.localtime
time.gmtimeon
```

### string

```
str.capitalize() 把字符串的第一个字符大写
str.center(width) 返回一个原字符串居中，并使用空格填充到width长度的新字符串
str.ljust(width) 返回一个原字符串左对齐，用空格填充到指定长度的新字符串
str.rjust(width) 返回一个原字符串右对齐，用空格填充到指定长度的新字符串
str.zfill(width) 返回字符串右对齐，前面用0填充到指定长度的新字符串
str.count(str,[beg,len]) 返回子字符串在原字符串出现次数，beg,len是范围
str.decode(encodeing[,replace]) 解码string,出错引发ValueError异常
str.encode(encodeing[,replace]) 解码string
str.endswith(substr[,beg,end]) 字符串是否以substr结束，beg,end是范围
str.startswith(substr[,beg,end]) 字符串是否以substr开头，beg,end是范围
str.expandtabs(tabsize = 8) 把字符串的tab转为空格，默认为8个
str.find(str,[stat,end]) 查找子字符串在字符串第一次出现的位置，否则返回-1
str.index(str,[beg,end]) 查找子字符串在指定字符中的位置，不存在报异常
str.isalnum() 检查字符串是否以字母和数字组成，是返回true否则False
str.isalpha() 检查字符串是否以纯字母组成，是返回true,否则false
str.isdecimal() 检查字符串是否以纯十进制数字组成，返回布尔值
str.isdigit() 检查字符串是否以纯数字组成，返回布尔值
str.islower() 检查字符串是否全是小写，返回布尔值
str.isupper() 检查字符串是否全是大写，返回布尔值
str.isnumeric() 检查字符串是否只包含数字字符，返回布尔值
str.isspace() 如果str中只包含空格，则返回true,否则FALSE
str.title() 返回标题化的字符串（所有单词首字母大写，其余小写）
str.istitle() 如果字符串是标题化的(参见title())则返回true,否则false
str.join(seq) 以str作为连接符，将一个序列中的元素连接成字符串
str.split(str=‘‘,num) 以str作为分隔符，将一个字符串分隔成一个序列，num是被分隔的字符串
str.splitlines(num) 以行分隔，返回各行内容作为元素的列表
str.lower() 将大写转为小写
str.upper() 转换字符串的小写为大写
str.swapcase() 翻换字符串的大小写
str.lstrip() 去掉字符左边的空格和回车换行符
str.rstrip() 去掉字符右边的空格和回车换行符
str.strip() 去掉字符两边的空格和回车换行符
str.partition(substr) 从substr出现的第一个位置起，将str分割成一个3元组。
str.replace(str1,str2,num) 查找str1替换成str2，num是替换次数
str.rfind(str[,beg,end]) 从右边开始查询子字符串
str.rindex(str,[beg,end]) 从右边开始查找子字符串位置 
str.rpartition(str) 类似partition函数，不过从右边开始查找
str.translate(str,del=‘‘) 按str给出的表转换string的字符，del是要过虑的字符
```

### subprocess <a href="#blogtitle16" id="blogtitle16"></a>

subprocess 模块允许我们启动一个新进程，并连接到它们的输入/输出/错误管道，从而获取返回值。

{% code overflow="wrap" %}
```
subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None, capture_output=False, shell=False, cwd=None, timeout=None, check=False, encoding=None, errors=None, text=None, env=None, universal_newlines=None)

args：表示要执行的命令。必须是一个字符串，字符串参数列表。
stdin、stdout 和 stderr：子进程的标准输入、输出和错误。其值可以是 subprocess.PIPE、subprocess.DEVNULL、一个已经存在的文件描述符、已经打开的文件对象或者 None。subprocess.PIPE 表示为子进程创建新的管道。subprocess.DEVNULL 表示使用 os.devnull。默认使用的是 None，表示什么都不做。另外，stderr 可以合并到 stdout 里一起输出。
timeout：设置命令超时时间。如果命令执行时间超时，子进程将被杀死，并弹出 TimeoutExpired 异常。
check：如果该参数设置为 True，并且进程退出状态码不是 0，则弹 出 CalledProcessError 异常。
encoding: 如果指定了该参数，则 stdin、stdout 和 stderr 可以接收字符串数据，并以该编码方式编码。否则只接收 bytes 类型的数据。
shell：如果该参数为 True，将通过操作系统的 shell 执行指定的命令。
```
{% endcode %}

Popen 是 subprocess的核心，子进程的创建和管理都靠它处理。

```
class subprocess.Popen(args, bufsize=-1, executable=None, stdin=None, stdout=None, stderr=None, 
preexec_fn=None, close_fds=True, shell=False, cwd=None, env=None, universal_newlines=False, 
startupinfo=None, creationflags=0,restore_signals=True, start_new_session=False, pass_fds=(),
*, encoding=None, errors=None)
```

```
args：shell命令，可以是字符串或者序列类型（如：list，元组）
bufsize：缓冲区大小。当创建标准流的管道对象时使用，默认-1。
0：不使用缓冲区
1：表示行缓冲，仅当universal_newlines=True时可用，也就是文本模式
正数：表示缓冲区大小
负数：表示使用系统默认的缓冲区大小。
stdin, stdout, stderr：分别表示程序的标准输入、输出、错误句柄
preexec_fn：只在 Unix 平台下有效，用于指定一个可执行对象（callable object），它将在子进程运行之前被调用
shell：如果该参数为 True，将通过操作系统的 shell 执行指定的命令。
cwd：用于设置子进程的当前目录。
env：用于指定子进程的环境变量。如果 env = None，子进程的环境变量将从父进程中继承。

```

## 2、python可变类型、不可变类型

属于可变类型的类型: List,Dictionary,Set.可变数据类型是指变量所指向的内存地址处的值是可以被改变的,也就是说可变类型在赋值的时候copy的是地址或者引用

属于不可变类型的类型: Number ,Str,tuple，不可变数据类型在第一次声明赋值声明的时候, 会在内存中开辟一块空间, 用来存放这个变量被赋的值, 而这个变量实际上存储的, 并不是被赋予的这个值, 而是存放这个值所在空间的内存地址, 通过这个地址, 变量就可以在内存中取出数据了

## 3、python参数传递的\*args和\*\*kwargs

`*args`和`**kwargs`一般是用在函数定义的时候。二者的意义是允许定义的函数接受任意数目的参数。也就是说我们在函数被调用前并不知道也不限制将来函数可以接收的参数数量。在这种情况下我们可以使用`*args`和`**kwargs`。

`*args`用来表示函数接收可变长度的**非关键字**参数列表作为函数的输入

`**kwargs`表示函数接收可变长度的**关键字**参数字典作为函数的输入。当我们需要函数接收**带关键字**的参数作为输入的时候，应当使用`**kwargs。`

