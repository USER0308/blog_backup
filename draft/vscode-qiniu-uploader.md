# vscode 上传图片到七牛插件

## vscode 上传图片到七牛的插件
vscode-paste-image-to-qiniu
https://github.com/favers/vscode-qiniu-upload-image

qiniu-upload-image
https://github.com/yscoder/vscode-qiniu-upload-image
用的是第二个, 不过应该两个都能用

注意:

```
    // 七牛图床域名
    "qiniu.domain": "http://***.com"
```
域名要以 http 开头, 否则无法上传图片

另外, 快捷键可能会有冲突, 需要自己修改

## vscode 预览图片失败
预览失败时提示隐藏了不安全的内容, 只需要点击这条提示, 在弹出框中选择显示所有不安全的内容即可