#  文件的上传下载功能

## 一、上传(基于tea)

1、页面代码：
```js
import React, { useState, useRef } from "react";
import { Upload, Button, Form, Modal } from "tea-component";

export default function UploadExample() {
  const [file, setFile] = useState(null);
  const [status, setStatus] = useState(null);
  const [percent, setPercent] = useState(null);

  const xhrRef = useRef(null);

  function handleStart(file, { xhr }) {
    setFile(file);
    setStatus("validating");
    xhrRef.current = xhr;
  }

  function handleProgress({ percent }) {
    setPercent(percent);
  }

  function handleSuccess(result) {
    console.log(result);
    setStatus("success");
  }

  function handleError() {
    setStatus("error");
    handleAbort();
    Modal.alert({
      type: "error",
      message: "错误",
      description: "请求服务器失败",
    });
  }

  function beforeUpload(file, fileList, isAccepted) {
    if (!isAccepted) {
      setStatus("error");
    }
    return isAccepted;
  }

  function handleAbort() {
    if (xhrRef.current) {
      xhrRef.current.abort();
    }
    setFile(null);
    setStatus(null);
    setPercent(null);
  }

  return (
    <Form.Control
      status={status}
      message="请上传 png 格式文件，大小 100KB 以内"
    >
      <Upload
        <!-- action="https://run.mocky.io/v3/68ed7204-0487-4135-857d-0e4366b2cfad" -->
        accept="image/png"
        maxSize={100 * 1024}
        onStart={handleStart}
        onProgress={handleProgress}
        onSuccess={handleSuccess}
        onError={handleError}
        beforeUpload={beforeUpload}
      >
        {({ open }) => (
          <Upload.File
            filename={file && file.name}
            percent={percent}
            button={
              status === "validating" ? (
                <Button onClick={handleAbort}>取消上传</Button>
              ) : (
                <>
                  <Button onClick={open}>
                    {status ? "重新上传" : "选择文件"}
                  </Button>
                  {status && (
                    <Button
                      type="link"
                      style={{ marginLeft: 8 }}
                      onClick={handleAbort}
                    >
                      删除
                    </Button>
                  )}
                </>
              )
            }
          />
        )}
      </Upload>
    </Form.Control>
  );
}
```
选择文件之后，即可在需要的地方拿到file这个文件对象，在需要的地方调用接口，将这个文件对象传给后台即可。

参考链接：https://tea-design.github.io/component/upload

2、后台：

## 二、下载
1、页面代码：
```html
<Button
  type="primary"
  onClick={() => {
      downloadFile();
  }}
  >
  下载 
</Button>
```

2、模拟a标签下载：
```js
  const downloadStringAsFile = (content, filename) => {
    // 创建隐藏的可下载链接
    var eleLink = document.createElement('a');
    eleLink.download = filename;
    eleLink.style.display = 'none';
    // 字符内容转变成blob地址
    var blob = new Blob([content], { type: 'text/csv,charset=UTF-8' });
    eleLink.href = URL.createObjectURL(blob); // 等价于下面这种写法
    // eleLink.href = 'data:text/csv;charset=utf-8,' + encodeURIComponent(content);

    // 触发点击
    document.body.appendChild(eleLink);
    eleLink.click();
    // 然后移除
    document.body.removeChild(eleLink);
  };
```
3、文件内容处理：
```js
const downloadFile = () => {
        // const res = {    // txt文件
        //   filename: new Date().getTime() + '.txt',
        //   filecontent: '所有的功夫感觉啊福建省地方u的繁花似锦横幅，广告速度回复觉得更好的风景更好今后的复古风'
        // };
        const res = {    // csv文件
          filename: new Date().getTime() + '.csv',       
          filecontent: [      // 经过后台处理的json字符串
            ["阿瑟费地方","是对方感动","收费金额发b","返回放入","凤凰大道"],
            ["阿瑟费地方","是对方感动","收费金额发b","返回放入","凤凰大道"],
            ["阿瑟费地方","是对方感动","收费金额发b","返回放入","凤凰大道"],
            ["阿瑟费地方","是对方感动","收费金额发b","返回放入","凤凰大道"],
          ]
        }
        const content = res.filecontent.map(item => {
          const substr2 = item.join().match(/\{.+\}/g);
          if (substr2 !== null) {
            const substr1 = substr2[0].replaceAll(',', '，');
            return item.join().replaceAll(substr2[0], substr1);
          }
          return item.join();
        }).join('\n');
        downloadStringAsFile(content, res.filename);
  };
```

## 三、html页面生成图片并下载到本地

1、html页面生成图片，需要用到html2canvas这个插件

安装：npm install --save html2canvas
```js
  import React,{ useRef,useState } from 'react';
  import html2canvas from 'html2canvas';

  const content = useRef()
  const htmltoimg = ()=>{
    const timestamp = (new Date()).valueOf()
    html2canvas(content.current, {
      allowTaint: false,
      useCORS: true,
    }).then(function (canvas) {
        const dataImg = new Image()
        dataImg.src = canvas.toDataURL('image/png')
        const alink = document.createElement("a");
        alink.href = dataImg.src;
        alink.download = `testImg${timestamp}.jpg`;
        alink.click();

        // 将生成的图片转为文件对象形式传给后台
        // function dataURLtoFile() {//将base64转换为文件  
        //  const arr = dataImg.src.split(','), mime = arr[0].match(/:(.*?);/)[1]
        // let bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);  
        //  while(n--){  
        //      u8arr[n] = bstr.charCodeAt(n);  
        //  }  
        //  return new File([u8arr], `xiaobaibai${timestamp}.jpg`, {type:mime});  
        // }

        // 发送请求到后台代码
        // const formData = new FormData()   
        // formData.append('file',dataURLtoFile());
        // service.upload(formData).then(res=>{
        //   console.log(res);
        // }) 
    });
  }

  return (
        <>
            <Content>
            {/* <Content.Header title='这是头部' />    工作台不需要 */}
                <Content.Body full>
                    <Card ref={content}>
                    <Card.Body>
                    （此处是你自己的页面代码page1）
                    </Card.Body>
                    </Card>
                </Content.Body>
            </Content>
        </>
    )
```

2、页面中调用 htmltoimg 这个方法，就会得到如下的图片：

![](../image/htmltoimg.jpg)

## 四、html页面生成图片并上传到cos

- 页面部分：同三(包括注释部分)。

- 后台部分：

1、安装 cos-nodejs-sdk-v5：npm install cos-nodejs-sdk-v5

2、后台config.default.ts中添加以下配置：

```js
 config.cos = {
    SecretId: 'AKIDzn7Bbk7VyIQHtfldbNuEI6lPoxbNPF0W',  //项目的cosId
    SecretKey:'3qVu0pOhqq17ww4MWTDaHiyxlRq7YnmU',  //项目的cos密码
    Bucket:'xiaobaibai-1303955107',   //项目的cos
    Region:'ap-guangzhou'   //地区
  };
```
controller下file.ts中代码：
```js
public async upload() {
        const { ctx, service } = this;
        const stream = await ctx.getFileStream();
        try {
            const res: any = await service.file.upload(stream.filename, stream);
            ctx.body = {
                ret: retcode.ok,
                data:res,
            };
        }catch (err) {
            ctx.body = {
                ret: retcode.err,
                message: err.message,
                stack: err.stack,
            };
        }
    }
```
service下file.ts中代码：
```js
const COS = require('cos-nodejs-sdk-v5');

export default class extends Service {
  async upload(key: string, stream: any): Promise<string | void> {
    return new Promise(
      (resolve, reject) => {
        const { SecretId, SecretKey, Bucket, Region } = this.config.cos;
        const cos = new COS({
          SecretId,
          SecretKey
        });
        cos.putObject({
          Bucket, /* 必须 */
          Region,    /* 必须 */
          Key: key,              /* 必须 */
          StorageClass: 'STANDARD',
          Body: stream,
          // Body: fs.createReadStream(stream), // 上传文件对象
          onProgress: function (progressData) {
            console.log(JSON.stringify(progressData));
          }
        }, function (err, data) { // data 类型为Object: {} 如果请求发生错误，则为空
          if (JSON.stringify(data) === "{}") {
            return reject(err);
          }
          console.log(JSON.stringify(data));
          return resolve(data);
        });
      }
    );
  }
}
```
