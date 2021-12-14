## Install and Deploy Serverless

### [此份文件的環境 :]
* OS: Ubuntu 20.04
* serverless --version:
> * Framework Core: 2.66.0  
> * Plugin: 5.5.1  
> * SDK: 4.3.0  
> * Components: 3.17.2

### [此份文件的練習題 :]

[AWS Lambda + Serverless Framework + Python](https://faun.pub/aws-lambda-serverless-framework-python-part-1-a-step-by-step-hello-world-4182202aba4a)


### 在 Ubuntu 上安裝 serverless
```
$ sudo apt install npm
$ sudo npm install -g serverless
$ sudo npm update -g serverless
$ serverless --version
```

使用 ```$ serverless --version``` 檢查版本時出現下面文字，表示要更新 Node.js

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/serverless_version_warning.png)


### 在 Ubuntu 上更新 Node.js

按照文章中的步驟去更新

[更新最新版本 Node.js 在 Ubuntu 20.04](https://blog.impochun.com/how-to-install-latest-nodejs-on-ubuntu/)

更新後再看一次
```$ serverless --version```

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/serverless_version.png)

----

### create virtual environment
```$ python3.7 -m venv /path/to/your/{virtual-env}```

### enable your python virtual environment
```$ source /path/to/your/{virtual-env}/bin/activate```



### Initializing our Project
```$ serverless create --template aws-python3 --name {project_name}```

> 下完後會多出 handler.py 與 serverless.yml 這兩個檔案

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/Initialize_project.png)


### serverless.yml 與 handler.py

```$ vim serverless.yml```
> provider 是指部屬時的環境設定  
> * 將 Python 的版本改成 3.7 
> * 指定 region
> * 設定執行 lambda 的 user role

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/serverless_yml.png)


> 在 AWS GUI 設定 lambda function 時需要指定哪個 user role 來執行

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/lambda_user.png)


handler.py 裡面放的是 lambda function

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/handler.png)


### AWS switch role
```$ assume-role {your-profile-name}```
> * 之前有設定好 bash 的指令別名，可以直接執行 assume-role 的指令  
>   ex: assume-role zyxel-camelot-dev  
> * 如果沒有設定 bash 或 zsh 的指令別名，要執行 `$ eval $(assume-role -duration 8h0m0s {your-profile-name})`

### depoly serverless

```$ serverless deploy```

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/deploy_without_profile.png)


* 在部屬中可以在 CloudFormatin 上看到 Status 是 CREATE_IN_PROGRESS
* 部屬完後 Status 是 UPDATE_COMPLETE
![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/cloudformation_create_in_progress.png)


### [補充 : Serverless 是如何部屬到 AWS?]

[AWS - Deploying](https://www.serverless.com/framework/docs/providers/aws/guide/deploying/)


serverless 的套件會將 serverless.yml 裡的 Fuctions, Events, Resources 轉換成 AWS CloudFormation 的樣版(ex: json 格式)
> 我們在 ~/.aws/config 設定 output 格式

* An AWS CloudFormation template is created from your serverless.yml.
* If a Stack has not yet been created, then it is created with no resources except for an S3 Bucket, which will store zip files of your Function code.
* The code of your Functions is then packaged into zip files.
* Zip files of your Functions' code are uploaded to your Code S3 Bucket.
* The CloudFormation Stack is updated with the new CloudFormation template.
* Each deployment publishes a new version for each function in your service.

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/deploy_without_profile.png)


* Serverless fetches the hashes for all files of the previous deployment (if any) and compares them against the hashes of the local files.
* Serverless terminates the deployment process if all file hashes are the same.

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/deploy_not_changed.png)


### [狀況 : 我已經做 assume-role 了，但在執行 $ serverless deploy 時發生錯誤]

錯誤訊息：The security token included in the request is expired
> 解決方法：  
> 將 `AWS_ACCESS_KEY_ID` 清空再執行一次 assume role(switch role)  
> ```$ unset AWS_ACCESS_KEY_ID```  
> ```$ assume-role {your-profile-name}```

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/deploy_expired.png)

