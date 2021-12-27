## install development environment

### [此份文件的環境 :]
* OS: Ubuntu 20.04
* serverless --version:
> * Framework Core: 2.66.0  
> * Plugin: 5.5.1  
> * SDK: 4.3.0  
> * Components: 3.17.2


> step1 ~ step5 是來自 Matt 寫的文  
> [Matt ENV_SETUP](https://github.com/zyxel-dc/camelot-backend/blob/develop/ENV_SETUP.md)


### 1. install python3.7 on ubunto 20.04
https://linuxize.com/post/how-to-install-python-3-7-on-ubuntu-18-04/
> Matt 的環境是 Ubuntu 18.04，而我的是 Ubuntu 20.4，但這邊步驟是一樣的

```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository ppa:deadsnakes/ppa
```

> 第三步驟是新增一個地方，讓 `apt-get update upgrade` 可以讓 Ubuntu 多找那個地方。就是一個網址讓 Ubuntu apt 可以抓到的意思

```
$ sudo apt install python3.7
$ sudo apt install python3.7-venv
$ sudo apt install python3.7-dev
$ sudo apt install python3-pip
$ python3.7 -m pip install pip
```

### 2. create virtual environment

```$ python3.7 -m venv /path/to/your/{virtual-env}```

### 3. enable your python virtual environment

```$ source /path/to/your/{virtual-env}/bin/activate```

### 4. aws development required

```
$ pip install pip --upgrade
$ pip install awscli
$ pip install botocore
$ pip install boto
$ pip install boto3
```

> awscli 可以安裝在 Ubuntu 本機上  
> 如果安裝失敗可以使用下面指令安裝  
> ```$ sudo apt-get install awscli```

> 檢查是否安裝成功  
> ```$ aws --version```

### 5. install project required python packages

```$ pip install -r /path/to/your/camelot-backend/apps/${app_folder}/requirements.txt```

### 6. 要設定 MFA with assume-role (在 Ubuntu 本機上設定)
> 資料來自 Redmine 的 Wiki 裡的文件
> [MFA with assume role](https://redmine.marketplace.zyxel.com/projects/digital-commerce/wiki/MFA_with_assume-role)


要使用 awscli 需要先與 AWS account 做綁定

那我們一般使用 AWS  圖形化介面時會透過 zyxel-entrance 的 account 做登入

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/AWS-GUI-login.png)

### 6-1. 在 root 底下建立 .aws 的資料夾 (在 Ubuntu 本機上設定)
```$ mkdir ~/.aws```

### 6-2. 建立並設定 credentails 檔案 (在 Ubuntu 本機上設定)
```
$ mkdir ~/.aws/credentials
$ vim  ~/.aws/credentials
```

* aws_access_key_id 與 aws_secret_access_key 需要另外找管理者拿
>  在下圖範例中指的是 Ada.Chen@zyxel.com 這個帳號的 aws_access_key_id 與 aws_secret_access_key


![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/credentials.png)


> 尋找 zyxel-entrance 的 account ID

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/credentials-2.png)


> 尋找 swtich role 後的 account ID (在此圖範例是 zyxel-camelot-dev 的 account)

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/credentials-3.png)


### 6-3. 建立並設定 config 檔案 (在 Ubuntu 本機上設定)

> 資料來自 Redmine 的 Wiki 裡的文件  
> [MFA with assume role](https://redmine.marketplace.zyxel.com/projects/digital-commerce/wiki/MFA_with_assume-role)

```$ vim  ~/.aws/config```

在 config file 中可以設定佈署的 region、輸出的格式

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/config.png)

### [補充] 透過 AWS CLI 設定 credentails、config 檔案

> 資料來自 AWS 官方文件  
> [Configuration basics](https://docs.aws.amazon.com/zh_tw/cli/latest/userguide/cli-configure-quickstart.html)  
> [Configuration and credential file settings](https://docs.aws.amazon.com/zh_tw/cli/latest/userguide/cli-configure-files.html)

```$ aws configure```

使用者需要輸入以下 4 項資訊：
* AWS Access Key ID
* AWS Secret Access Key
* Default region name
* Default output format


AWS CLI 會將剛剛輸入的敏感憑證資訊存放在名為 `credentials` 的本機檔案中(`AWS Access Key ID` 與 `AWS Secret Access Key`)

較不敏感組態選項，則存放在名為 `config` 的本機檔案中(`Default region name` 與 `Default output format`)

AWS 會將這些資訊存放在名為 `default` 的設定檔 (設定集合)


### [補充] 可以指定 profile name 去設定 credentails、config 檔案
```$ aws configure --profile profile_name ```

> 在此範例中使用 `aws configure --profile cat` 並輸入資訊
![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/aws-configure-1.png)

> `Default region name` 與 `Default output format` 確實存在 `config` file 中
![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/aws-configure-2.png)


> `AWS Access Key ID` 與 `AWS Secret Access Key` 確實存在 `credentials` file 中
![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/aws-configure-3.png)


### 6-4. 安裝 assume-role 的套件 (在 Ubuntu 本機上設定)

> 資料來自 Redmine 的 Wiki 裡的文件  
> [MFA with assume role](https://redmine.marketplace.zyxel.com/projects/digital-commerce/wiki/MFA_with_assume-role)

assume-role 所做的事情與我們在 AWS GUI 上操作 switch role 的動作一樣

我們要先安裝此套件才可以做到 switch role 的動作

```$ sudo apt-get install git```
> 這個步驟是在安裝是跳出來要在執行的步驟(還要使用新環境再次確認)

```
$ sudo apt install golang
$ go get -u github.com/remind101/assume-role
$ sudo ln /home/{your_ubuntu_account}/go/bin/assume-role /usr/bin/assume-role
```

> golang 是可以讓 Ubuntu 連到 GitHub 上下載檔案  
> (go get 可以從網路下載及安裝指定的 package 以及相關的依賴代碼)

### 6-5. 設定 assume-role alias (在 Ubuntu 本機上設定)

**[方法一 :] 透過 eval 去執行 assue-role**

執行 assume-role：`$ eval $(assume-role -duration 8h0m0s zyxel-camelot-dev)`
> eval 會去執行 $(assume-role -duration 8h0m0s zyxel-camelot-dev) 的結果
> 最後一個參數就是 profile name

**[方法二 :] 設定 assume-role alias**

> [How to Create Bash Aliases](https://linuxize.com/post/how-to-create-bash-aliases/)

跟 git 一樣，我們可以將指令做別名的動作，讓指令更加簡潔

有 bash 和 zsh 可以使用，在此文件使用的是 bash

```
$ vim ~/.bashrc
$ assume-role -duration 8h0m0s zyxel-camelot-dev
```
> 在 ~/.bashrc 的最後一行加上這下方程式  
> `function assume-role { eval $( $(which assume-role) $@); } `   
> 設定好別名後 assume role 這個指令會去執行 `$ eval $(assume-role -duration 8h0m0s test)`  

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/assumed_role_MFA.png)


### 7. 檢查是否 assume role 成功
```$ env```

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/assumed_role.png)

### 8. 執行 AWS CLI
上面步驟都設定好 AWS 認證後，我們要檢查是否可以使用 AWS

```$ aws s3 ls```

### [補充] 如果要切換 assume role 的話要先把 `AWS_ACCESS_KEY_ID` 清空再做 assume role

```$ unset AWS_ACCESS_KEY_ID```

### [補充] 開啟 Vim 外部貼上功能

[[VIM] 複製貼上——開啟系統剪貼簿的支援](https://clay-atlas.com/blog/2020/04/20/vim-cn-note-how-to-copy-and-paste-to-external-program/)
