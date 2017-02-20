# Fastlane deliver

使用 fast deliver

## #1

在一个空白目录使用

```shell
fastlane deliver init
```

会根据输入的 ITC 用户名和密码, App Identifier 创建DevliverFile, 用于身份验证和app的确认

## #2

上传metadata和截图需要准备metadata和screenshots目录,目录如下

![](https://github.com/Flowever/Tech/raw/master/public/img/Screen Shot 2017-02-20 at 11.12.28 AM.png)

支持的语言列表

```shell
en-US, en-CA, fi, ru, zh-Hans, nl-NL, zh-Hant, en-AU, id, de-DE, sv, ko, ms, pt-BR, el, es-ES, it, fr-CA, es-MX, pt-PT, vi, th, ja, fr-FR, da, tr, en-GB
```

截图会自动根据图片尺寸上传,同尺寸的图片顺序按照文件名称的字母序决定

注意: url需要填写可以访问的网址, fastlane会检查链接的有效性

## #4
在 DevliverFile 中指定 AppIcon, 审核信息

配置 Example

```Shell
app_icon "./metadata/AppIcon.png" # App Icon

# 提交审核信息:加密, idfa 等
submission_information({    
    export_compliance_encryption_updated: false,
    export_compliance_uses_encryption: false,
    content_rights_contains_third_party_content: false,
    add_id_info_uses_idfa: false
})

# 应用审核小组的联系信息 app 审核信息
app_review_information(
  first_name: "name",
  last_name: "name",
  phone_number: "+86 400 820 8820",
  email_address: "email@example.com",
  demo_user: "测试账号用户名",
  demo_password: "测试账号密码",
  notes: "备注"
)
```


## #3
上传命令

```shell
# 只上传 metadata, 不上传 ipa 和截图, 不提交审核
fastlane deliver --skip_binary_upload --skip_screenshots

# 只上传截图, 不上传 ipa 和 metadata, 不提交审核
fastlane deliver --skip_binary_upload --skip_metadata

# 只上传 metadata 和 截图, 不上传 ipa, 不提交审核
fastlane deliver --skip_binary_upload

# 只上传指定 ipa, 并提交审核
deliver --ipa "App.ipa" --skip_screenshots --skip_metadata --submit_for_review

# 上传 metadata, 截图 和指定 ipa, 并提交审核
deliver --ipa "App.ipa" --submit_for_review

# 下载 metadata
fastlane deliver download_metadata

# 下载截图
fastlane deliver download_screenshots
```
