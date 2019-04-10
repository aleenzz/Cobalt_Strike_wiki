# 0x00 Cobalt Strike

这几章基本是官方文档我修改了一些,这章没有编程基础可能看起来很吃力，可以先看看sleep2.1文档，当然我在 `0x02 Listeners`中简单的讲了下每行代码是干什么的可以先看这个再回头看其他的。

Aggressor脚本是Cobalt Strike中的拓展,大多数Cobalt Strike对话框和功能都是作为独立模块编写的,它们向Aggressor Script引擎公开了一些接口。

内部脚本default.cna定义了默认的Cobalt Strike体验,这个脚本定义了Cobalt Strike的工具栏按钮，弹出菜单，它还为大多数Cobalt Strike事件格式化输出.


## 键盘快捷键

脚本可以创建键盘快捷键,使用bind绑定键盘快捷键,这个例子显示了 Hello World！在按住Ctrl和H的对话框中

```
bind Ctrl + H { 
    show_message（“Hello World！”）; 
}
```

## 弹出菜单

脚本也可以添加到Cobalt Strike的菜单结构或重新定义它，popup关键字为弹出钩子构建菜单层次结构。

这是定义Cobalt Strike帮助菜单的代码

```
popup help {
        item("&Homepage", { url_open("https://www.cobaltstrike.com/"); });
        item("&Support",  { url_open("https://www.cobaltstrike.com/support"); });
        item("&Arsenal",  { url_open("https://www.cobaltstrike.com/scripts?license=" . licenseKey()); });
        separator();
        item("&System Information", { openSystemInformationDialog(); });
        separator();
        item("&About", { openAboutDialog(); });
}

```

简单的说就是`popup `定义一个菜单栏 然后用 `item` 构建他的菜单目录，`separator()` 是一个分隔符函数，注册新菜单需要用 `menubar("&xxx", "xxx");` 。

```
popup pgraph {
        menu "&Layout" {
        item "&Circle"    { graph_layout($1, "circle"); }
        item "&Stack"     { graph_layout($1, "stack"); }
        menu "&Tree" {
            item "&Bottom" { graph_layout($1, "tree-bottom"); }
            item "&Left"   { graph_layout($1, "tree-left"); }
            item "&Right"  { graph_layout($1, "tree-right"); }
            item "&Top"    { graph_layout($1, "tree-top"); }
        }
        
        item "&None" { graph_layout($1, "none"); }
    }
}

```

`menu` 来构建主菜单中的子菜单 

## 自定义输出

Aggressor Script中的set关键字定义了如何格式化事件并将其输出呈现给用户

```
set EVENT_SBAR_LEFT {
    return "[" . tstamp(ticks()) . "] " . mynick();
}

set EVENT_SBAR_RIGHT {
    return "[lag: $1 $+ ]";
}

```

上面的代码定义了Cobalt Strike的事件日志（视图 - >事件日志）中状态栏的内容,此状态栏的左侧显示当前时间和您的昵称。右侧显示了Cobalt Strike客户端和团队服务器之间的消息往返时间。
你可以覆盖Cobalt Strike默认脚本中的任何设置选项，使用你关注的事件的定义创建自己的文件，加载到Cobalt Strike中，Cobalt Strike将使用你对内置定义的定义。

## 活动

使用on关键字为事件定义处理程序,当Cobalt Strike连接到团队服务器并准备就绪事件将触发。

```
on ready {
    show_message("Ready for action!");
}

```

Cobalt Strike为各种情况生成事件。使用* meta-event观看Cobalt Strike发生的所有事件

```
on * {
    local('$handle $event $args');
    
    $event = shift(@_);
    $args  = join(" ", @_);
    
    $handle = openf(">>eventspy.txt");
    writeb($handle, "[ $+ $event $+ ] $args");
    closef($handle);
}
```



# 0x01 Data Model

Cobalt Strike的团队服务器存储您的主机，服务，凭据和其他信息,它还广播此信息并将其提供给所有客户。

## API

使用＆data_query函数查询Cobalt Strike的数据模型,此功能可以访问Cobalt Strike客户端维护的所有状态和信息,使用＆data_keys获取您可能查询的不同数据的列表,此示例查询Cobalt Strike数据模型中的所有数据，并将其导出到文本文件

```
command export {
    local('$handle $model $row $entry $index');
    $handle = openf(">export.txt");
    
    foreach $model (data_keys()) {
        println($handle, "== $model ==");
        println($handle, data_query($model));
    }
    
    closef($handle);
    
    println("See export.txt for the data.");
}

```

其中CS给我们提供了很多API 后期我们讲到 ,数据模型中的每个函数返回一个数组,数组中包含条目字典。


# 0x02 Listeners

## Listeners API 

Aggressor Script会聚当前连接到的所有团队服务器的侦听器信息,这样可以轻松地将会话传递给另一个团队服务器。要获取所有侦听器名称的列表,请使用`＆listeners`函数。如果您只想使用本地侦听器,请使用`＆listeners_local`,在与`listener_info()`功能解决了一个侦听器名称其配置信息,此示例将所有侦听器及其配置转储到Aggressor Script控制台.

```
command listeners {        
    local('$name $key $value');
    foreach $name (listeners()) {
        println("== $name == ");
        foreach $key => $value (listener_info($name)) {
            println("$[20]key : $value");
        }
    }
}

```

首先使用command 定义个指令 listeners 你可以把他当成定义函数的形势，无伤大雅，然后 `local` 分析指定的字符串并将字符串中的所有变量声明为局部变量
`foreach` 循环`listeners()` 获取所以的监听器名称 再循环打印每个`listener_info($name) ` 的数据，可能说的不是很清楚用python简单的表示下。

```
listeners = {'http':{'payload':'windows/beacon_http/reverse_http'},
             'smb':{'payload':'windows/beacon_smb/reverse_smb'}
            }
for name in listeners.items():
    print name
    for key in listeners[name]:
        print key,listeners[name][key]

```

我们执行Aggressor Script脚本后可以看到

```
aggressor> listeners
== http == 
payload              : windows/beacon_http/reverse_http
port                 : 8090
host                 : 192.168.59.134
useragent            : Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0; MASP)
dns                  : 0
name                 : http
beacons              : 192.168.59.134
status               : success

```

其他都是一些functions的讲解，后面再说


# 0x03 Beacon

## Metadata

Cobalt Strike为每个Beacon分配会话ID,此ID是随机数。Cobalt Strike将任务和元数据与每个Beacon ID相关联,使用`＆beacons`查询所有当前Beacon会话的元数据。使用`＆beacon_info`查询特定Beacon会话的元数据，这是一个转储有关每个Beacon会话的信息的脚本，这个其实跟Listeners差不多

```
command beacons {
    local('$entry $key $value');
    foreach $entry (beacons()) {
        println("== " . $entry['id'] . " ==");
        foreach $key => $value ($entry) {
            println("$[20]key : $value");
        }
        println();
    }
}
```

## Aliases

您可以使用alias关键字定义新的Beacon命令,这是一个在Beacon控制台中打印Hello World的hello别名，就是定义个beacon的命令。

```
alias hello {
       blog($1, "Hello World!");
}

```

可以把参数传输给 alias定义的名称 其中 $0 是别名和参数，$1 是别名输入的Beacon的ID 具体可以参考他的Function表。

```
alias saywhat {
    blog($1, "My arguments are: " . substr($0, 8) . "\n");
}

```


# 0x04 文末

本章简单翻译了下文档 大概讲了下Aggressor的功能

### 本文如有错误，请及时提醒，以免误导他人
