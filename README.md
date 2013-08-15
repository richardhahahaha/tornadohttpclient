TornadoHTTPClient 对tornado.curl_httpclient.CurlAsyncHTTPClient的封装, 支持cookie

## 安装
```bash
python setup.py install
```

## 教程
### GET
TornadoHTTPClient的get方法可以发起一个get请求
```python
from tornadohttpclient import TornadoHTTPClient

# 实例化
http = TornadoHTTPClient()

# 发出get请求
http.get("http://www.linuxzen.com")

# 开始主事件循环
http.start()
```

### POST
TornadoHTTPClient的post方法可以发起一个post请求

### 读取响应
上面仅仅发出了请求, 但是我们无法读取GET请求回来的数据, 我们可以使用一个回调来读取响应
```python
from tornadohttpclient import TornadoHTTPClient

http = TornadoHTTPClient()

def callback(response):
    print response.body
    http.stop()

http.get("http://www.linuxzen.com", callback = callback)
http.start()
```

通过`callback`关键字参数我们可以传进一个回调函数, 当请求成功时会调用此函数, 并给此函数传递一个与`urllib2.urlopen`返回一样的reponse实例

### 上传文件
`upload`方法可以上传文件, 其接受一个url和文件的field和文件路径, 还有其他post参数
```python
from tornadohttpclient import TornadoHTTPClient

http = TornadoHTTPClient()
def callback(response):
    print("打开图片链接", end = " ")
    print(response.effective_url)
    http.stop()

http.upload("http://paste.linuxzen.com", "img", "img_test.png",
                    callback = callback)
http.start()
```

### 给callback传递参数
有时候callback可能需要访问局部变量, 可以通过 `args`和`kwargs`关键字参数, 将`callback`的参数传递给`get`/`post`方法, `args`参数将会在`response`参数之前被传递,
`args`参数类型应当是一个元组, `kwargs`参数类型应当是一个字典
```python
from tornadohttpclient import TornadoHTTPClient

http = TornadoHTTPClient()

def callback(times, response):
    print response.body
    print times

    if times == 9:
        http.stop()

for i in range(10):
    http.get("http://www.linuxzen.com", callback = callback, args = (i, ))

http.start()
```

### 发送延迟请求
有时我们需要延迟几秒也发送请求或每隔几秒就发送一个请求, `get`/`post`方法的`delay`关键字参数可以解决, `delay`参数接受一个单位为秒的数字, 并延迟`delay`秒后发起请求
```python
from tornadohttpclient import TornadoHTTPClient

http = TornadoHTTPClient()

def callback(response, times):
    print response.body
    if times < 9:
        # 延迟10秒发送此请求
        http.get("http://www.linuxzen.com", callback = callback, args = (times + 1, ), delay = 10)
    else:
        http.stop()

http.get("http://www.linuxzen.com", callback = callback, args = (1, ))
http.start()
```

### 给请求传递参数
TornadoHTTPClient 的 `get`/`post`方法的第二个参数`params`可以定义请求时传递的参数`params`的类型为字典或者`((key, value), )`类型的元组或列表,例如使用百度搜索`TornadoHTTPClient`
```python
from tornadohttpclient import TornadoHTTPClient

http = TornadoHTTPClient()

def callback(response):
    print response.body
    http.stop()

http.get("http://www.baidu.com/s", (("wd", "tornado"),), callback = callback)
http.start()
```

以上也使用与POST方法, 比如登录网站
```python
from tornadohttpclient import TornadoHTTPClient

http = TornadoHTTPClient()

def callback(response):
    print response.body
    http.stop()

http.post("http://ip.or.domain/login", (("username", "cold"), ("password", "pwd")), callback = callback)

http.start()
```


## 设置User-Agent
```python
from tornadohttpclient import TornadoHTTPClient

http = TornadoHTTPClient()
http.set_user_agent( "Mozilla/5.0 (X11; Linux x86_64)"\
                " AppleWebKit/537.11 (KHTML, like Gecko)"\
                " Chrome/23.0.1271.97 Safari/537.11")

def callback(response):
    print response.body
    http.stop()

http.get("http://www.linuxzen.com", headers=headers, callback = callback)
```

### 指定HTTP头
TornadoHTTPClient 的`get`/`post`方法的 `headers`关键字参数可以自定额外的HTTP头信息, 参数类型为一个字典

指定User-Agent头

```python
from tornadohttpclient import TornadoHTTPClient

http = TornadoHTTPClient()

def callback(response):
    print response.body
    http.stop()

headers = dict((("User-Agent",
                "Mozilla/5.0 (X11; Linux x86_64)"\
                " AppleWebKit/537.11 (KHTML, like Gecko)"\
                " Chrome/23.0.1271.97 Safari/537.11"), ))

http.get("http://www.linuxzen.com", headers=headers, callback = callback)
```

### 使用代理
TornadoHTTPClient 的`set_proxy`方法可以设置代理, 其接受四个参数, 分别是代理的 主机名/ip 代理的端口 代理用户名 代理用户密码, 如无认证只传前两个即可, `unset_proxy`可以取消代理
```python
from tornadohttpclient import TornadoHTTPClient

http = TornadoHTTPClient()

def callback(response):
    print response.body
    http.unset_proxy()
    http.stop()

http.set_proxy("127.0.0.1", 8087)
http.get("http://shell.appspot.com", callback = callback)
http.start()
```

### Cookie
TornadoHTTPClient会自动记录和装载Cookie, 可以通过 TornadoHTTPClient实例属性 cookie 获取Cookie
