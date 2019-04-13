# 0x00 profile语法分析

以 `backoff.profile` 为例来讲解

```
#
# Backoff POS Malware
#
# This profile takes steps to dress up the POST side of Beacon's C2 to
# look like Backoff. The GET side is left generic.
#
# Indicators from:
#   http://blog.spiderlabs.com/2014/07/backoff-technical-analysis.html
#   https://gsr.trustwave.com/topics/backoff-pos-malware/backoff-malware-overview/
#
# Author: @armitagehacker
#
# 
set sample_name "Backoff POS Malware";

set sleeptime "30000"; # use a ~30s delay between callbacks
set jitter    "10";    # throw in a 10% jitter

set useragent "Mozilla/5.0 (Windows NT 6.1; rv:24.0) Gecko/20100101 Firefox/24.0";

# the relevant indicators
http-post {
    set uri "/windebug/updcheck.php /aircanada/dark.php /aero2/fly.php /windowsxp/updcheck.php /hello/flash.php";

    client {
        header "Accept" "text/plain";
        header "Accept-Language" "en-us";
        header "Accept-Encoding" "text/plain";
        header "Content-Type" "application/x-www-form-urlencoded";

        id {
            netbios;
            parameter "id";
        }

        output {
            base64;
            prepend "&op=1&id=vxeykS&ui=Josh @ PC&wv=11&gr=backoff&bv=1.55&data=";
            print;
        }
    }

    server {
        output {
            print;
        }
    }
}

# No information on backoff use of GET, so generic GET request.
http-get {
    set uri "/updates";

    client {
        metadata {
            netbiosu;
            prepend "user=";
            header "Cookie";
        }
    }

    server {
        header "Content-Type" "text/plain";

        output {
            base64;
            print;
        }
    }
}

```

注释符号 `#`  选择赋值 `set` 用来设置一些程序的默认值 语句以`;` 结束,类似JavaScript.

代码中 `set sleeptime "30000";`  即为设置心跳时间为30000毫秒，`set jitter    "10";` 为默认抖动系数（0-99%）

`set useragent "Mozilla/5.0 (Windows NT 6.1; rv:24.0) Gecko/20100101 Firefox/24.0";` 设置 user-agent 其中的http-get与http-post基本为固定语法了。

```
http-get {
    set uri "/updates";

    client {
        metadata {

        }
    }

    server {
        header "";

        output {

        }
    }
}

```

为了更好的看懂这些，以下是官网语法文档。



# 0x01 文档

配合官方文档写了下,其中很多用不到，没关系需要什么看什么,长文字直接用的机翻了大概意思差不多。

## Data Transform Language

数据转换也就是CS内置的几种编码


Statement | Action |  Inverse  
-|-|-
append "string"	 | Append "string"	| Remove last LEN("string") characters
base64	| Base64 Encode	| Base64 Decode
base64url	| URL-safe Base64 Encode |	URL-safe Base64 Decode
mask	| XOR mask w/ random key	| XOR mask w/ same random key
netbios	| NetBIOS Encode 'a'	| NetBIOS Decode 'a'
netbiosu	| NetBIOS Encode 'A'	| NetBIOS Decode 'A'
prepend "string" |	Prepend "string" |	Remove first LEN("string") characters

数据转换语句可以任意数量顺序组合,以终止语句结束,在转换中只能使用一个终止语句

Statement | What 
-|-|
header "header" |Store data in an HTTP header
parameter "key" | Store data in a URI parameter
print   | Send data as transaction body
uri-append | Append to URI

终止语句将转换后的数据储存到Http头中，参数终止语句将转换的数据 print来最后发送这些编码的数据

print是 `http-get.server.output`，`http-post.server.output` 和 `http-stager.server.output` 的终止语句配合上文代码可以看出。

其他块使用 `header`，`parameter`，`print`和`uri-append` `termination`语句,如果在`http-post.client.output`上使用`header` `parameter` `uri append` `termination`语句，beacon会将其响应分块到一个合理的长度，以适应事务的一部分。

## Strings

Beacon的Profile语法可以多个地方使用 Strings

Value | Special Value 
-|-|
"\n"  |  Newline character
"\r"   | Carriage Return
"\t"   | Tab character
"\u####"  |  A unicode character
"\x##"  | A byte (e.g., \x41 = 'A')
"\\"   | \

## Options

Beacon的默认值,分为全局和本地,全局更改Beacon的设置，本地用于特定事务。

Option | Context | Default Value | Changes 
-|-|-|-|
amsi_disable  | null  |    false  | (Attempt to) disable AMSI for execute-assembly, powerpick, and psinject
dns_idle    | null |    0.0.0.0 | IP address used to indicate no tasks are available to DNS Beacon; Mask for other DNS C2 values
dns_max_txt  | null |   252   | Maximum length of DNS TXT responses for tasks
dns_sleep    | null |   0  | Force a sleep prior to each individual DNS request. (in milliseconds)
dns_stager_prepend   | null  |   null  |    Prepend text to payload stage delivered to DNS TXT record stager
dns_stager_subhost   | null |   .stage.123456.    | Subdomain used by DNS TXT record stager.
dns_ttl  | null |   1    |  TTL for DNS replies
host_stage   | null    |   true     |  Host payload for staging over HTTP, HTTPS, or DNS. Required by stagers.
jitter     | null  |   0     | Default jitter factor (0-99%)
maxdns     | null  |   255   | Maximum length of hostname when uploading data over DNS (0-255)
pipename     | null   |    msagent_##   |  Name of pipe to use for SMB Beacon's peer-to-peer communication. ## is replaced with a number unique to your team server.
pipename_stager  | null   |    status_##    |  Name of pipe to use for SMB Beacon's named pipe stager. ## is replaced with a number.
sample_name   | null   |   My Profile   |  The name of this profile (used in the Indicators of Compromise report)
sleeptime     | null   |   60000     | Default sleep time (in milliseconds)
spawnto_x86   | null   |   %windir%\syswow64\rundll32.exe   |  Default x86 program to open and inject shellcode into
spawnto_x64   | null     | %windir%\sysnative\rundll32.exe  |  Default x64 program to open and inject shellcode into
tcp_port     | null    |   4444     |  TCP Beacon listen port
uri   | http-get,http-post    |  [required option]  |   Transaction URI
uri_x86   | http-stager      | null |  x86 payload stage URI
uri_x64   | http-stager      | null | x64 payload stage URI
useragent    |null  |     Internet Explorer (Random)   |  Default User-Agent for HTTP comms.
verb     |  http-get,http-post   | GET,POST     |  HTTP Verb to use for transaction


以上都是简单的英语我也就懒的翻译了,使用URI可以将多个URI 指定为一个空格分隔的字符串,CS会自己处理。


## Beacon HTTP Transaction

HTTP请求 参数

Request | Component | Block | Data 
-|-|-|-|
http-get   |   client  |  metadata   |   Session metadata
http-get    |  server  |  output  |  Beacon's tasks
http-post  |   client  |  id  |  Session ID
http-post   |  client  |  output  |  Beacon's responses
http-post   |  server   | output  |  Empty
http-stager  | server |   output  |  Encoded payload stage

#### HTTP Staging

Beacon是一个分阶段的payload，有效负载由stager下载并注入内存，在目标内存中有Beacon之前HTTP GET和HTTP POST不会生效。
Malleable C2的http-stager块可自定义HTTP分段过程。

```
http-stager {

      set uri_x86 "/get32.gif";

      set uri_x64 "/get64.gif";

```

uri_x86选项设置URI下载x86的payload,uri_x64选项设置URI下载64位的payload 。



## Self-signed Certificates with SSL Beacon

HTTPS Beacon在其通信中使用HTTP Beacon的指示符,Malleable C2配置文件还可以指定Beacon C2服务器的自签名SSL证书的参数。

```
https-certificate {

      set CN       "bobsmalware.com";

      set O        "Bob's Malware";

}

```

证书参数

Option | Example | Description 
-|-|-|
C   |  US   | Country
CN   | beacon.cobaltstrike.com  | Common Name; Your callback domain
L   |  Washington   | Locality
O   |  Strategic Cyber LLC  | Organization Name
OU   | Certificate Department  |  Organizational Unit Name
ST  |  DC  |  State or Province
validity   |   365  | Number of days certificate is valid for


## Valid SSL Certificates with SSL Beacon

可以选择将有效SSL证书与Beacon一起使用。使用Malleable C2配置文件指定Java密钥库文件和密码。此密钥库必须包含证书的私钥，根证书，任何中间证书以及SSL证书供应商提供的域证书。
Cobalt Strike在与Malleable C2配置文件相同的文件夹中找到Java Keystore文件。

```
https-certificate {

      set keystore "domain.store";

      set password "mypassword";

}

```


Option | Example | Description 
-|-|-|
Option | Example | Description
keystore  |  domain.store   | Java Keystore file with certificate information
password  |  mypassword | The password to your Java Keystore

以下是创建用于Cobalt Strike的Beacon的有效SSL证书的步骤：

1.使用keytool程序创建Java密钥存储文件。这个程序会询问“你的姓名是什么？” 确保使用完全权威的域名来响应Beacon服务器。另外，请确保记下密钥库密码,你以后会需要它。

`$ keytool -genkey -keyalg RSA -keysize 2048 -keystore domain.store`

2.使用keytool生成证书签名请求（CSR）,您将向您的SSL证书供应商提交此文件,他们将验证您的身份并颁发证书,有些供应商比其他供应商更容易和便宜。

`$ keytool -certreq -keyalg RSA -file domain.csr -keystore domain.store`

3.导入SSL供应商提供的Root和任何中间证书。

`$ keytool -import -trustcacerts -alias FILE -file FILE.crt -keystore domain.store`

4.最后，您必须安装域证书。

`$ keytool -import -trustcacerts -alias mykey -file domain.crt -keystore domain.store`

就是这样就生成Cobalt Strike的Beacon一起使用的Java Keystore文件。

## Code Signing Certificate

>Attacks -> Packages -> Windows Executable and Windows Executable (S)

提供签署可执行文件或DLL文件的选项,需要 代码签名证书和私钥指定Java Keystore文件

```
code-signer {
            set keystore "keystore.jks";
            set password "password";
            set alias    "server";
}

```

Option | Example | Description 
-|-|-|
alias  | server | The keystore's alias for this certificate
digest_algorithm  |  SHA256  |The digest algorithm
keystore   | keystore.jks   | Java Keystore file with certificate information
password  |  mypassword | The password to your Java Keystore
timestamp |  false  | Timestamp the file using a third-party service
timestamp_url |  http://timestamp.digicert.com  | URL of the timestamp service


## PE and Memory Indicators

Malleable C2 stage http-stager 控制Beacon如何加载到内存中并编辑Beacon DLL的内容。

```
stage {
            set userwx "false"; 
            set compile_time "14 Jul 2009 8:14:00";
            set image_size_x86 "512000";
            set image_size_x64 "512000";
            set obfuscate "true";
 
            transform-x86 {
                        prepend "\x90\x90";
                        strrep "ReflectiveLoader" "DoLegitStuff";
            }
            transform-x64 {
                        # transform the x64 rDLL stage
            }
 
            stringw "I am not Beacon!";
}

```

当接受后将字符串添加到beacon dll的.rdata部分，string命令添加一个以zero-terminated的字符串。stringw命令添加了一个宽（utf-16le编码）字符串,
Transform-x86和Transform-X64阻止PAD和Transform Beacon的反射DLL阶段。这些块支持三个命令：prepend、append和strrep.
prepend命令在beacon的反射dll之前插入一个字符串,append命令在beacon-reflective dll后面添加一个字符串,确保预先准备好的数据是阶段体系架构（x86、x64）的有效代码,c2lint程序没有对此进行检查,strrep命令替换beacon反射dll中的字符串。

stage块接受Beacon DLL内容的选项:

Option | Example | Description 
-|-|-|
checksum   |  0   | The CheckSum value in Beacon's PE header
cleanup  |false  |  Ask Beacon to attempt to free memory associated with the Reflective DLL package that initialized it.
compile_time   |  14 July 2009 8:14:00    | The build time in Beacon's PE header
entry_point | 92145  |  The EntryPoint value in Beacon's PE header
image_size_x64  | 512000 |  SizeOfImage value in x64 Beacon's PE header
image_size_x86  | 512000 |  SizeOfImage value in x86 Beacon's PE header
module_x64  | xpsservices.dll | Same as module_x86; affects x64 loader
module_x86  | xpsservices.dll | Ask the x86 ReflectiveLoader to load the specified library and overwrite its space instead of allocating memory with VirtualAlloc.
name   |  beacon.x64.dll   |The Exported name of the Beacon DLL
obfuscate   | false  |  Obfuscate the Reflective DLL's import table, overwrite unused header content, and ask ReflectiveLoader to copy Beacon to new memory without its DLL headers.
rich_header   | null  | Meta-information inserted by the compiler
sleep_mask   |false  |  Obfuscate Beacon, in-memory, prior to sleeping
stomppe  |true   |  Ask ReflectiveLoader to stomp MZ, PE, and e_lfanew values after it loads Beacon payload
userwx  | false   | Ask ReflectiveLoader to use or avoid RWX permissions for Beacon DLL in memory

## Cloning PE Headers

Cobalt Strike的Linux软件包,包括一个工具peclone，用于从dll中提取头文件并将其显示为一个随时可用的阶段块：

`./peclone [/path/to/sample.dll]`

## In-memory Evasion and Obfuscation

使用stage块的prepend命令来破坏分析，该分析扫描内存段的前几个字节以查找注入的dll的迹象。如果使用特定于工具的字符串检测代理，请使用strrep命令更改它们。

如果strrep不够，请将sleep_mask设置为true。这将引导信标在进入睡眠状态之前在记忆中模糊自己。在休眠之后，信标会将自己的模糊处理为请求和处理任务。SMB和TCP信标在等待新连接或等待来自其父会话的数据时会使它们自己变得模糊。

决定您希望在内存中看起来有多像一个DLL。如果您希望方便检测，请将stomppe设置为false。如果您想在内存中稍微混淆信标dll，请将stomppe设置为true。如果你想挑战，将“模糊”设置为“真”。此选项将采取许多步骤来模糊信标阶段和内存中DLL的最终状态。

将userwx设置为false以询问beacon的加载器以避免rwx权限。具有这些权限的内存段将吸引分析师和安全产品的额外关注。

默认情况下，Beacon的加载程序使用virtualloc分配内存。模块踩踏是一种替代方法。将module_x86设置为一个大约是beacon有效载荷本身两倍大的dll。Beacon的x86加载程序将加载指定的dll，在内存中查找其位置并覆盖它。这是一种在内存中定位信标的方法，Windows将其与磁盘上的文件关联。您要驻留的应用程序不需要您选择的DLL，这一点很重要。模块_x64选项的情况相同，但它会影响x64信标。

如果您担心在内存中初始化beacon dll的beacon阶段，请将cleanup设置为true。此选项将在不再需要信标阶段时释放与之关联的内存。

## Process Injection

Malleable C2配置文件中的进程注入块可以注入内容并控制进程注入行为

```
process-inject {
            set min_alloc "16384";
            set startrwx "true";
            set userwx "false";
 
            transform-x86 {
                        prepend "\x90\x90";
            }
            transform-x64 {
                        # transform x64 injected content
            }
 
            disable "CreateRemoteThread";
}

```

transform-x86和transform-x64阻止Beacon注入的PAD内容。这些块支持两个命令：prepend和append

prepend命令在插入的内容之前插入一个字符串。append命令在注入的内容之后添加一个字符串。确保预先准备好的数据是注入内容体系结构（x86、x64）的有效代码。c2lint程序没有对此进行检查。

disable语句是避免在beacon的进程注入例程中使用某些API的提示。您可以禁用：sethreadcontext、createRemoteThread和rtlcreateUserThread。请注意，当您禁用这些调用时，可能会在Beacon的进程注入例程中引入可避免的失败。c2lint命令会发出一些警告。

process-inject块接受几个控制Beacon中的过程注入的选项

Option | Example | Description 
-|-|-|
min_alloc  |   4096    |  Minimum amount of memory to request for injected content
startrwx   |   true  |    Use RWX as initial permissions for injected content. Alternative is RW.
userwx  |  false |  Use RWX as final permissions for injected content. Alternative is RX.

# 0x02 文末


### 本文如有错误，请及时提醒，以免误导他人
