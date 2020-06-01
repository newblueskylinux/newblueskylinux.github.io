---
layout: post
title:  "将虚拟机导入到aws的ec2"
categories: AWS 
tags: awscli vmvare ec2
author: 郑禹
---

* content
{:toc}
---

	本地服务器迁移上云是我们运维人员在运维工作中很常见的一个工作。
	实体机镜像打包很困难，但是从vmvare虚拟机中打包就买那么苦难了，让我们来看看是如何完成的。

## 一、导出OVA模板

	首先，需要将虚拟机关机，选定虚拟机，点击导出OVF模板
	
<img src="http://zhengyu1992.cn/img/ova1.png">





	将OVA文件存放到本地目录中
	
<img src="http://zhengyu1992.cn/img/ova2.png">

##	二、创建服务角色（此操作只需要做一次）

	1.在当前目录下创建一个名为 trust-policy.json 的文件，在文件中写入以下内容（此文件是通过json库为创建的角色授权）

```sh
{
   "Version": "2012-10-17",
   "Statement": [
      {
         "Effect": "Allow",
         "Principal": { "Service": "vmie.amazonaws.com" },
         "Action": "sts:AssumeRole",
         "Condition": {
            "StringEquals":{
               "sts:Externalid": "vmimport"
            }
         }
      }
   ]
}

```

	2.使用 create-role 命令创建名为 vmimport 的角色，并向 VM Import/Export 提供对该角色的访问权，如果json文件不在当前目录下需要加上文件所在的绝对路径。

```sh
aws iam create-role --role-name vmimport --assume-role-policy-document "file://trust-policy.json"
```

	3.在当前目录下创建一个名为 role-policy.json 的文件，在文件中写入以下内容（此文件是通过json库为vmimport角色指定存储桶），其中，第一个arn:aws-cn:s3:::jxs-new为从本地导入OVA文件所用的的存储桶，第二个arn:aws-cn:s3:::jxs-new为存储所导出AMI映像的存储桶

```sh
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Action":[
            "s3:GetBucketLocation",
            "s3:GetObject",
            "s3:ListBucket" 
         ],
         "Resource":[
            "arn:aws-cn:s3:::jxs-new",
            "arn:aws-cn:s3:::jxs-new/*"
         ]
      },
      {
         "Effect":"Allow",
         "Action":[
            "s3:GetBucketLocation",
            "s3:GetObject",
            "s3:ListBucket",
            "s3:PutObject",
            "s3:GetBucketAcl"
         ],
         "Resource":[
            "arn:aws-cn:s3:::jxs-new",
            "arn:aws-cn:s3:::jxs-new/*"
         ]
      },
      {
         "Effect":"Allow",
         "Action":[
            "ec2:ModifySnapshotAttribute",
            "ec2:CopySnapshot",
            "ec2:RegisterImage",
            "ec2:Describe*"
         ],
         "Resource":"*"
      }
   ]
}
```

	(这一步一般不需要做）4.如果需要针对许可证配置附加到 AMI，需要向 role-policy.json 文件中增加以下内容

```sh
{
  "Effect":"Allow",
  "Action":[
    "license-manager:GetLicenseConfiguration",
    "license-manager:UpdateLicenseSpecificationsForResource",
    "license-manager:ListLicenseSpecificationsForResource"
  ],
  "Resource":"*"
}
```

	5.将我们上面创建的json权限配置文件应用到所创建的vmimport角色，如果json文件不在当前目录下需要加上文件所在的绝对路径

```sh
aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document "file://trust-policy.json"
```

##	三、导入OVA文件到EC2的AMI映像

	1.在当前目录下创建一个名为 containers.json 的文件，在文件中写入以下内容（此文件是通过json库为创建的角色授权）
	
	S3Bucket为存储桶的名称
	
	my-server-vm.ova为ova的文件名
	
```sh
[
  {
    "Description": "My Server OVA",
    "Format": "ova",
    "UserBucket": {
        "S3Bucket": "jxs-new",
        "S3Key": "my-server-vm.ova"
    }
}]
```

	2.将OVA文件导入到EC2的AMI映像，如果json文件不在当前目录下需要加上文件所在的绝对路径。

```sh
aws ec2 import-image --description "win_server2012" --disk-containers "file://containers.json"
```

	3.命令执行成功后可通过执行以下命令查看正在导入的任务进程
	
```sh
aws ec2 describe-import-image-tasks
```

	如果状态显示completed则认为导入成功

<img src="http://zhengyu1992.cn/img/task1.png">

##	四、aws控制台完成AMI文件生成ec2实例

	1.登录到EC2控制台点击AMI可查看导入成功的映像
	
<img src="http://zhengyu1992.cn/img/task2.png">

	2.用导出的AMI映像启动一台EC2实例，根据需求选择实例类型，启动开机即可完成

---