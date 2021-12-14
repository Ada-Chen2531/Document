## Sending Emails from AWS Lambda Using SES service

### [此份文件的環境 :]
* OS: Ubuntu 20.04
* serverless --version:
> * Framework Core: 2.66.0  
> * Plugin: 5.5.1  
> * SDK: 4.3.0  
> * Components: 3.17.2

### [此份文件的練習題 :]

[Sending Emails from AWS Lambda Using SES service](https://faun.pub/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-3-sending-emails-from-ad4119abca3c)

### create virtual environment
```
$ python3.7 -m venv /path/to/your/{virtual-env}
$ cd {virtual-env}
$ mkdir app
```
> `$ python3.7 -m venv ./using-ses`   
> `$ cd using-ses/`  
> `$ mkdir app`

### enable your python virtual environment
```$ source /path/to/your/{virtual-env}/bin/activate```
> `$ source ./bin/activate`

### Initializing our Project
```$ serverless create --template aws-python3 --name {project_name}```

> `$ cd app`  
> `$ serverless create --template aws-python3 --name using-ses`

### AWS switch role
```$ assume-role {your-profile-name}```
> * 之前有設定好 bash 的指令別名，可以直接執行 assume-role 的指令  
>   ex: assume-role zyxel-camelot-dev  
> * 如果沒有設定 bash 或 zsh 的指令別名，要執行 `$ eval $(assume-role -duration 8h0m0s {your-profile-name})`

### 在 serverless.yml 加入寄信的權限與 function
* we should add permissions to the serverless.yml file
* Our application should be granted access to SES

``` yml
provider:
  iamRoleStatements:
    - Effect: Allow
      Action:
        - ses:SendEmail
        - ses:SendRawEmail
      Resource: "*"  
```

* We need also to define our function in the serverless.yml

``` yml
functions:
  sendEmail:
    handler: handler.sendEmail
    description: This function will send an email
    events:
      - http:
          path: send-email
          method: post
          integration: lambda
          cors: true
          response:
            headers:
              "Access-Control-Allow_Origin": "'*'"
```

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/ses_serverless.png)


### 在 handler.py 加入 function

* 這邊也使用 boto3 去取得 credentials file 的 access_key 和 secret_key

``` py
import json
import boto3
import os

session = boto3.Session()

credentials = session.get_credentials()
access_key = credentials.access_key
secret_key = credentials.secret_key
region_name = 'us-west-1'

def sendEmail(event, context):
    # We are going to read the message from the data send using a POST
    data = event['body']
    name = data ['name']
    source = data['source']
    subject = data['subject']
    message = data['message']
    destination = data['destination']

    _message = "Message from: " + name + "\nEmail: " + source + "\nMessage content: " + message
    
    client = boto3.client('ses' )
        
    response = client.send_email(
        Destination={
            'ToAddresses': [destination]
            },
        Message={
            'Body': {
                'Text': {
                    'Charset': 'UTF-8',
                    'Data': _message,
                },
            },
            'Subject': {
                'Charset': 'UTF-8',
                'Data': subject,
            },
        },
        Source=source,
    )
    return _message + str(region_name)

```

### deploy serverless

``` $ serverless deploy```

### send an email
* Then we can use a simple CURL command to send the POST data and send the email
* 記得替換成自己的 email

```$ curl -X POST -d "name=ada" -d "source=ada.chen@zyxel.com.tw" -d "subject=This is me" -d "destination=ada.chen@zyxel.com.tw" -d "message=this is my message" https://jclh84kfra.execute-api.us-west-1.amazonaws.com/dev/send-email```

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/send_email_failed.png)

> 會執行失敗是因為該 email 還沒有在 AWS SES 上認證


### verify your email by using the AWS SES Console

* 到 AWS console --> SES service --> click the "Verified identities" --> click the "Create identity" button

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/aws_console_ses_1.png)


![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/aws_console_ses_2.png)


* 一開始設定完會顯示 Unverified

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/aws_console_ses_3.png)

* 到自己的信箱上點擊 URL 去認證

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/aws_console_ses_4.png)

* 認證成功

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/aws_console_ses_5.png)


### send an eamil again

```$ curl -X POST -d "name=ada" -d "source=ada.chen@zyxel.com.tw" -d "subject=This is me" -d "destination=ada.chen@zyxel.com.tw" -d "message=this is my message" https://jclh84kfra.execute-api.us-west-1.amazonaws.com/dev/send-email```

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/send_email_successfully-1.png)

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/send_email_successfully-2.png)

