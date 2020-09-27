# 使用 PHP 与 JavaScript 上传文件

## Reference

- [利用FormData对象实现AJAX文件上传功能及后端实现](https://juejin.im/post/6844903539379011592)
- [PHP解决跨域问题，你会用哪种方法](https://zhuanlan.zhihu.com/p/139998926)

-------------------------------

## 初衷

上传文件是项目中许多地方都要用到的一个通用功能。所以这里暂时使用 PHP 为后端处理上传文件，前端只做上传操作。

假定后端上传文件接口：<http://localhost/upload>，只允许`POST`上传。

## 前端代码

这里只做一个简单的单/多文件上传示例，使用原生 HTML 和 JavaScript 代码，如下：

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <div id="uploads">
        <input type="file" name="multiple[]" id="multiple" multiple="multiple">
        <br>
        <input type="file" name="single" id="single">
        <br>
        <button id="submit">submit</button>
    </div>

    <script>
        function post(server, param = '') {
            let request = new XMLHttpRequest();
            request.open('POST', server);
            request.send(param);
            request.onreadystatechange = function () {
                if (request.readyState === 4 && request.status === 200) {
                    let data = JSON.parse(request.response);
                    console.log(data);
                }
            };
        }

        let submit = document.getElementById('submit');
        if (submit) {
            submit.addEventListener('click', function () {
                let formdata = new FormData();

                let multipleFiles = document.getElementById('multiple').files;
                // console.log(multipleFiles);

                // 如果是多个文件则循环
                for (let index = 0; index < multipleFiles.length; index++) {
                    const element = multipleFiles[index];

                    formdata.append('multiple[]', element);// 标签名应该是 name[]
                }

                // 如果是单文件
                let singleFile = document.getElementById('single').files[0];
                console.log(singleFile);
                formdata.append('single', singleFile);// 标签名应该是 name
                // console.log(formdata);

                let server = 'http://localhost/upload';// 文件上传接口
                post(server, formdata);
            })
        }

    </script>
</body>

</html>
```

这里使用的是 JavaScript 的`FormData`类和它的`append()`方法构建表单传递数据。

## 后端处理

这里主要是处理前端跨域问题：

```php
    /**
     * 上传文件接口-对外接口
     * 要求指定上传文件的标签名，否则上传所有文件
     * 返回上传文件的所有信息
     *
     * @Route("/upload", name="upload")
     *
     * @param Request $request
     * @param FileUploader $fileUploader
     * @return Response
     */
    public function upload(Request $request, FileUploader $fileUploader, string $labelName = ''): Response
    {
        header("Access-Control-Allow-Origin:*");
        header('Access-Control-Allow-Methods:POST');

        $response = $this->uploadFile($request, $fileUploader, $labelName);

        return new JsonResponse($response);
    }
```

这里也只是做了简单的跨域访问策略，更完善的机制以后再学习。
（上传文件接口具体之后再补充）
