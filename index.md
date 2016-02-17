# OnlineJudgeOpenAPI文档

为了方便与 Virtual Judge 进行集成，开放了获取题目详细信息、提交代码和获取代码运行结果三个API。

在使用API之前，请先申请appkey，在个人设置页面可以看到，如果没有申请过，请联系OJ的管理员在后台开通。

##API说明
所有的返回值都是`{"code": <int>, data: <Object>}`的形式，只有code为0的时候代表正常返回了，data为数据内容。其余code表示出现错误，data为错误提示。

所有的POST请求和响应都是json格式的，POST请求的`Content-Type`确保为`application/json`。

##获取题目详细信息
**request** `GET` `/api/open/problem/?appkey=<string>&problem_id=<int>`
**response**

```js
{
    "code": 0,
    "data": {
        // 题目的id
        "id": 1,
        // 样例输入和输出
        "samples": [
            {
                "input": "1 1",
                "output": "2"
            },
            {
                "input": "1 1",
                "output": "2"
            },
            {
                "input": "1 -1",
                "output": "0"
            }
        ],
        // 标签
        "tags": [
            {
                "id": 1,
                "name": "简单"
            }
        ],
        // 创建用户
        "created_by": {
            "username": "root"
        },
        // 题目
        "title": "A + B Problem",
        // 描述 HTML格式
        "description": "<p>请计算两个整数的和并输出结果。</p><p>注意不要有不必要的输出，比如&quot;请输入 a 和 b 的值: &quot;，示例代码见隐藏部分。</p>",
        // 输入说明
        "input_description": "两个用空格分开的整数.",
        // 输出说明
        "output_description": "两数之和",
        // 提示 没有提示就是空字符串
        "hint": "测试题目",
        // 创建时间
        "create_time": "2015-09-02T13:02:26Z",
        // 最后修改时间 如果没有修改过，就是NULL
        "last_update_time": "2016-02-02T03:43:34.244046Z",
        // 时间限制 单位ms
        "time_limit": 1000,
        // 内存限制 单位M
        "memory_limit": 512,
        // 总共提交次数
        "total_submit_number": 1128,
        // 总共ac次数
        "total_accepted_number": 521,
        // 难度 1-3 简单到难
        "difficulty": 1,
        // 题目来源
        "source": "经典题目"
    }
}
```

##提交题目
**request** `post` `/api/open/submission/`

```js
{
    // appkey <string>
    "appkey": "example_appkey", 
    // 代码 <string>
    "code": "example code",
    // 语言 1:C 2:C++ 3:Java <int>
    "language": 1,
    // 题目id <int>
    "problem_id": 1
}
```

**response**
提交代码后，服务器立即返回，并异步判题。

```js
{
    "code": 0,
    "data": {
        // 提交id
        "submission_id": "4e49416e087f79fd3d0822b1899d601c"
    }
}
```

要注意的是，每个用户都有自己的提交频率限制。开源代码中，默认使用的TokenBucket进行的限制，每个用户默认有50个token，然后每分钟可以创建2个token，但是也是50个token封顶，每提交一道题就消耗一个token。开始的50个token可以保证一定时间的并发需求，如果超过频率限制将返回错误和需要等待的时间。

##获取提交结果

**request** `GET` `/api/open/submission/?appkey=<string>&submission_id=<string>`

**response**

```js
{
    "code": 0,
    "data": {
        "id": "9d4610ef9ae6b30e588c650891ba6858",
        "result": 0,
        "create_time": "2016-02-16T03:54:10Z",
        "language": 1,
        // info可能是None或者字符串
        // 在编译错误和系统错误的时候info为错误详情，可能会很长，其余的情况为一个json字符串
        "info": "[{\"cpu_time\": 0, \"exit_status\": 0, \"signal\": 0, \"output_md5\": \"33d6548e48d4318ceb0e3916a79afc84\", \"flag\": 0, \"result\": 0, \"memory\": 7602176, \"real_time\": 4}, {\"cpu_time\": 0, \"exit_status\": 0, \"signal\": 0, \"output_md5\": \"e4da3b7fbbce2345d7772b0674a318d5\", \"flag\": 0, \"result\": 0, \"memory\": 7602176, \"real_time\": 2}]"
    }
}
```

result的对应关系

```js
{
    "accepted": 0,
    "runtime_error": 1,
    "time_limit_exceeded": 2,
    "memory_limit_exceeded": 3,
    "compile_error": 4,
    "format_error": 5,
    "wrong_answer": 6,
    "system_error": 7,
    "waiting": 8
}
```


