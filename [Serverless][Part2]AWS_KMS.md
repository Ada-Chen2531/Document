## Using AWS KMS with Lambda to Store & Read Sensitive Data & Secrets

### [此份文件的環境 :]
* OS: Ubuntu 20.04
* serverless --version:
> * Framework Core: 2.66.0  
> * Plugin: 5.5.1  
> * SDK: 4.3.0  
> * Components: 3.17.2

### [此份文件的練習題 :]

[Using AWS KMS with Lambda to Store & Read Sensitive Data & Secrets](https://faun.pub/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-2-using-aws-kms-with-9bdad3381024)

### create virtual environment
```
$ python3.7 -m venv /path/to/your/{virtual-env}
$ cd {virtual-env}
$ mkdir app
```
> `$ python3.7 -m venv ./using-kms`   
> `$ cd using-kms/`  
> `$ mkdir app`

### enable your python virtual environment
```$ source /path/to/your/{virtual-env}/bin/activate```
> `$ source ./bin/activate`

### Initializing our Project
```$ serverless create --template aws-python3 --name {project_name}```

> `$ cd app`  
> `$ serverless create --template aws-python3 --name using-kms`

### AWS switch role
```$ assume-role {your-profile-name}```
> * 之前有設定好 bash 的指令別名，可以直接執行 assume-role 的指令  
>   ex: assume-role zyxel-camelot-dev  
> * 如果沒有設定 bash 或 zsh 的指令別名，要執行 `$ eval $(assume-role -duration 8h0m0s {your-profile-name})`


### Install AWS SSM(Secure Secrets Manager)

[Install amazon-ssm-agent on Ubuntu](https://snapcraft.io/install/amazon-ssm-agent/ubuntu)

> `$ sudo apt update`  
> `$ sudo apt install snapd`  
> (在執行上面指令後，有訊息印出可以使用此指令自動移除一些軟體: `$ sudo apt autoremove`)  
> `$ sudo snap install amazon-ssm-agent --classic`

### Let’s start by creating a key to use later
```$ aws kms create-key```
>  create key 當下會有以下 output

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/create_key.png)

### 將剛剛建立好的 key 透過 AWS Systems Manager Parameter 加密並存進一個明碼的變數中
* With AWS Systems Manager Parameter Store, you can create Secure String parameters, which are parameters that have a plaintext parameter name and an encrypted parameter value. 
* Parameter Store uses AWS KMS to encrypt and decrypt the parameter values of Secure String parameters

```$ aws ssm put-parameter --name "my_password" --value "change_me" --type SecureString --key-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx```

### 我們可以透過 AWS SSM 的指令去看 "my_password" 裡「加密」後的 value
```$ aws ssm get-parameter --name "my_password"```
![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/encrypted_value.png)

### 我們可以透過 AWS SSM 的指令去看 "my_password" 裡「解密」後的 value
```$ aws ssm get-parameter --name "my_password"  --with-decryption```
![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/decrypted_value.png)

### 將 function 加入 handler.py
> 這邊會使用到 boto3 去取得 credentials 的資訊   
> 使用方式請參考下方資料  
> [read credentials from aws by boto 3](https://stackoverflow.com/questions/49735257/read-credentials-from-aws-by-boto-3)

``` py
import json
import boto3
import os

session = boto3.Session()

credentials = session.get_credentials()
access_key = credentials.access_key
secret_key = credentials.secret_key
region_name = 'us-west-1'

ssm_client = session.client('ssm')

def getSecret(event, context):
    my_password = ssm_client.get_parameter(Name='my_password', WithDecryption=True)
    return str(my_password)
```

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/kms_handler.png)

### 將 function 加入 serverless.yml

``` yml
functions:
  getSecret:
    handler: handler.getSecret
    description: This function will return a secret stored using KMS
    events:
      - http:
          path: get-secret
          method: get
          integration: lambda
          cors: true
          response:
            headers:
              "Access-Control-Allow_Origin": "'*'"
```

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/kms_serverless.png)

### Deploy the project
```$ aws deploy```
> deploy 後會得到 end point URL

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/kms_deploy.png)


### this is the URL we can visit to get the secret

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/kms_result.png)


### [補充 : 當初在 deploy 後遇到權限問題]

``` json
"An error occurred (AccessDeniedException) when calling the GetParameter operation: User: arn:aws:sts::xxxxxxxxxxxxx:assumed-role/DefaultLambdaRole/using-kms-dev-getSecret is not authorized to perform: ssm:GetParameter on resource: arn:aws:ssm:us-west-1:xxxxxxxxxxxxx:parameter/my_password because no identity-based policy allows the ssm:GetParameter action"
```

root cause: DefaultLambdaRole 沒有開通 SSM 的權限，請高荃開通後就可以使用了
