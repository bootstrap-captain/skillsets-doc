# Typora

- [官网链接](https://typora.io/)
- 一个比较方便的书写md文件的软件，包含mac和windows版本

## 图片上传

- 用typora书写的文档，最终上传到了github上，文档中的图片，希望通过typora进行上传到阿里云服务器上

###  picgo下载

- [官网下载](https://github.com/Molunerfinn/picgo/releases)，下载安装完毕后，在mac的隐私设置中放行
- 配置好对应的阿里云的bucket内容

![image-20250409160537241](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250409160537241.png)

### typora

- 必须先切换到中文，才有PicGo.app的选项

![image-20250409160746390](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250409160746390.png)

# IP

```BASH
ipconfig getifaddr en0
```

# sftp功能

```bash
 # mac的上传和下载
 - put source target   # 上传
 - get source target   # 下载
```

# 云服务器

## 1. 服务器相同IP，本地连接

```bash
cd /Users/shuzhan/.ssh
rm -rf known_hosts
```

![image-20250409161735225](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250409161735225.png)
