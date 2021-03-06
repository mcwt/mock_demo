
## 安装

```sh
pip install easy-mock
```



## 使用

**查看版本号**

```sh
$ easy_mock --version
2021.2.25rc4
```



**查看帮助**

```sh
$ easy_mock -h
usage: easy_mock [-h] [-V] [-p PORT] [-pb2y TO_YAML] [-pb2j TO_JSON]
                 [pb_source_file]

Generate mock service according to the YAML/JSON file in the current directory

positional arguments:
  pb_source_file        Need converted pb source file.

optional arguments:
  -h, --help            show this help message and exit
  -V, --version         show version
  -p PORT, --port PORT  Port that needs mock service.
  -pb2y TO_YAML, --pb-to-yaml TO_YAML
  -pb2j TO_JSON, --pb-to-json TO_JSON
```



**示例**



新建并切换到目录

```sh
$ mkdir test
$ cd test
```



编写YAML/YAM文件(easy_mock会加载当前目录下所有以.yml/.yaml结尾的文件)

```yaml
apis:
  - url: login/mcw # 接口路径
    method: POST # 接口方法
    request_schema: # 请求jsonschem 不传则不做输入合法性校验
        {
          "type": "object",
          "properties": {
            "username": {
              "type": "string"
            },
            "password": {
              "type": "string"
            },
          },
          "required": [
            "username",
            "password"
          ]
        }
    response_schema: # 响应jsonschem, 根据schema生成mock数据  response_schema 和 defined_data_list必须有一个匹配
        {
          "type": "object",
          "properties": {
            "code": {
              "type": "integer", # 数据类型
              "maximun": 100, # 数据范围 最大
              "minimun": 1 # 数据范围 最小
            },
            "msg": {
              "type": "string"
            },
            "token": {
              "type": "string"
            },
          },
          "required": [   # 若指定required, 则每次必定返回 code、 msg字段, token字段则随机返回
            "code",
            "msg"
          ]
        }
    defined_data_list: # 自定义返回 如果请求体于list中的body匹配， 则返回对应的response  response_schema 和 defined_data_list必须有一个匹配
        [
          {
            body:{"username":"edison", "password":"123"},
            response:{"code":-1, "msg":"密码输入不正确"}
          },
          {
            body:{"username":"lily", "password":"123"},
            response:{"code":-2, "msg":"用户名不存在"}
          },
          {
            body:{"username":"root", "password":"123"},
            response:{"code":1, "msg":"登录成功", "token":"5lCadRru(ADn2IE!$LV%x%JF3JNmz*Nf5nFieUG!r((&esi2CLI$jb!227Lh"}
          },
          {
            body:{"username":"lily"},
            response:{"code":-1, "msg":"密码是必填的"}
          },
          {
            body:{"password":"123"},
            response:{"code":-1, "msg":"用户名是必填的"}
          }
        ]
        
  # 最精简写法
  - url: login/mcw # 接口路径
    method: GET # 接口方法
    defined_data_list: # 自定义返回 如果请求体于list中的body匹配， 则返回对应的response
      [
        {
          body:{"username":"edison", "password":"123"},
          response:{"code":-1, "msg":"密码输入不正确"}
        }
      ]

```



运行

```sh
$ easy_mock -p 9000
 * Serving Flask app "easy_mock.core" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:9000/ (Press CTRL+C to quit)

```



调用mock接口

```sh
根据 response schema 返回随机数

$ curl -X POST 'http://172.20.25.168:9000/login/mcw' -d '{"username":"zhangsan", "password": "123"}' | python -m json.tool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   173  100   131  100    42   7440   2385 --:--:-- --:--:-- --:--:--  7705
{
    "code": 93,
    "msg": "qwWtdNaSuTbDYzSAODvBGvrrYaUPkXejjSdWmcnVtxKJsuwIPzFwGzSFBFPiKqAXGjUPClpVUKzXFVsFFNUPC",
    "token": "nLqGauEqZotIsq"
}


匹配自定义返回值

curl -X POST 'http://172.20.25.168:9000/login/mcw' -d '{"username":"root", "password": "123"}' | python -m json.tool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   153  100   115  100    38   6477   2140 --:--:-- --:--:-- --:--:--  6764
{
    "code": 1,
    "msg": "登录成功",
    "token": "5lCadRru(ADn2IE!$LV%x%JF3JNmz*Nf5nFieUG!r((&esi2CLI$jb!227Lh"
}
	
```





定义 json schema

```yaml
修改 response schema:

      response_schema: 
        {
          "type": "object",
          "properties": {
            "code": {
              "type": "integer", 
              "maximun": 1, # 修改数据范围 为 0-1
              "minimun": 0 # 修改数据范围 为 0-1
            },
            "msg": {
              "type": "string"
              "maxLength": 10, # 修改string长度为 5-10
              "minLength": 5 # 修改string长度为 5-10
            },
            "token": {
              "type": "string"
            },
          },
          "required": [  
            "code",
            "msg"
          ]
        }
```



调用mock接口

```sh
$ curl -X POST 'http://172.20.25.168:9000/login/mcw' -d '{"username":"zhangsan", "password": "123"}' | python -m json.tool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   182  100   140  100    42   7931   2379 --:--:-- --:--:-- --:--:--  8235
{
    "code": 1,
    "msg": "Nlgedsc",
    "token": "sImDahqhaSKuosjXbFlcTAanzvV"
}
```







## 自定义扩展



在当前目录下新建python文件 `processor.py`

```sh
$ touch processor.py
$ vim processor.py

def test(req, resp): 

    resp["username"] = req.get("username")
   
    return resp

```



在YAML/YAM文件接口详情中新增`setup` or `teardown`方法

```yaml
apis:
  login/mcw: 
    name: 用户登录
    desc: 用户登录成功，接口会返回一个token
    method: POST 
    body_mode: json
    setup: # setup 接受一个参数, 对请求题做前置操作, 例如加解密等
    teardown: test # teardown两个参数, 对请求体和响应体做后置操作, 例如响应体包含请求体中某一字段
      
    # setup、teardown 均可不填或留空
```





调用mock接口

```sh
$ curl -X POST 'http://172.20.25.168:9000/login/mcw' -d '{"username":"zhangsan", "password": "123"}' | python -m json.tool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   136  100    94  100    42   5135   2294 --:--:-- --:--:-- --:--:--  5222
{
    "code": 2,
    "msg": "gmBiSDVlixdNqntqciZHkyHwsLglpw",
    "token": "ZeyZODPPxnd",
    "username": "zhangsan"
}
```



