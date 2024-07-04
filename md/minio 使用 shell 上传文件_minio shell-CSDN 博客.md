> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Myron_007/article/details/134523444)

#### minio 使用 shell 上传文件

*   [前言](#_1)
*   [1. 编写调用脚本](#1__4)
*   [2. 测试脚本上传](#2_44)
*   [3. 候选脚本](#3_51)

前言
--

业务场景需要实现，[服务器](https://so.csdn.net/so/search?q=%E6%9C%8D%E5%8A%A1%E5%99%A8&spm=1001.2101.3001.7020)文件上传至存储服务。一种方式是安装 minio 的 linux 客户端，另一种方式是通过调用 minio 的 api 接口实现文件上传。后一种方式不需要依赖 minio 的客户端使用起来有一定便利性。本文介绍通过 minio api 接口的上传文件的脚本写法及用法

1. 编写调用脚本
---------

```
#!/bin/bash

# 检查参数是否正确
if [ "$#" -ne 2 ]; then
    echo "usage: $0 <bucket-name> <file-path>"
    exit 1
fi

# 设置MinIO信息
minio_url="http://192.168.100.100:7000"
minio_access_key="minio"
minio_secret_key="minio@admin"
minio_bucket="$1"

# 设置文件名和路径
file_path="$2"
file_

# 生成 HTTP 头
http_date=$(date -u "+%a, %d %h %Y %H:%M:%S GMT")
http_content_md5=$(openssl md5 -binary < $file_path | base64)
http_content_type=$(file -b --mime-type $file_path)
http_request_path="/$minio_bucket/$file_name"
http_request_body=""

http_request="PUT\n$http_content_md5\n$http_content_type\n$http_date\n$http_request_path"
http_signature=$(echo -en "${http_request}" | openssl sha1 -binary -hmac "${minio_secret_key}" | base64)

# 使用 cURL 和 MinIO API 上传文件
curl -X PUT \
  -H "Date: $http_date" \
  -H "Content-Type: $http_content_type" \
  -H "Content-MD5: $http_content_md5" \
  -H "Authorization: AWS $minio_access_key:$http_signature" \
  --upload-file "$file_path" \
  "$minio_url/$minio_bucket/$file_name"
```

2. 测试脚本上传
---------

*   第一个参数为存储桶名称 (minio 上请提前创建好)
*   第二个参数为文件路径

```
./upload_to_minio_new.sh my-bucket aaa.txt
```

3. 候选脚本
-------

```
#!/bin/bash

# usage: ./minio-upload my-bucket my-file.zip

bucket=$1
file=$2
host=192.168.100.100:7000
s3_key='minio'
s3_secret='minio@admin'
resource="/${bucket}/${file}"
content_type='application/zip'
date=$(date -R)
filesize=$(stat -c%s "$file")

_signature="PUT\n\n${content_type}\n${date}\n${resource}"
signature=$(echo -en "${_signature}" | openssl sha1 -hmac "${s3_secret}" -binary | base64)

curl -v --fail \
    -X PUT \
    --data-binary "@$file" \
    --header "Host: ${host}" \
    --header "Date: ${date}" \
    --header "Content-Type: ${content_type}" \
    --header "Content-Length: ${filesize}" \
    --header "Authorization: AWS ${s3_key}:${signature}" \
    http://${host}${resource}
```