### [此份文件的環境 :]
* region: us-west-1
* camelot environment: dev
* key: Ada-us-west-1.pem
* [VitrulBox 網卡設定](https://github.com/Ada-Chen2531/Document/blob/main/VirtualBox_Network_Setting.md)

### 參考文件
[devops-manager](https://github.com/zyxel-dc/devops-manager/tree/master/ansible/camelot)


### Add SSH Keys into GitHub

[SSH Keys for GitHub](https://jdblischak.github.io/2014-09-18-chicago/novice/git/05-sshkeys.html)

### Add github key and provision key for ssh forwarding
```
$ eval $(ssh-agent)
$ ssh-add ~/.ssh/account-region-specific-ec2-key
$ ssh-add ~/.ssh/your-github-private-key
```

### If assume-role is installed, we can export STS token before ansible deployment

``` $ assume-role -duration 8h0m0s zyxel-camelot-dev```


### modify and execute env.sh
> location: devops-manager-master/ansible/camelot/env.sh  
* modify
> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/env_sh.jpg)

* execute
> ``` $ source env.sh```

### 檢查要部屬的 region 有多少的 AZs(Availability Zones)

```$ aws ec2 describe-availability-zones --region region-name```
> ex: 在此份文件使用的是 us-west-1，該 region 有 2 個 AZs，分別是 us-west-1a 與 us-west-1c   
> `$aws ec2 describe-availability-zones --region us-west-1`  

> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/AZs.jpg)

### modify vpc.yml

*  因為每個 region AZ 數量不同，要做修改  
*  原本的 vpc.yml playbook 是有 3 個 AZs，因此建了 3 個 private subnet 給各自的 AZ 使用
*  在此文件的 us-west-1 region 只有兩個 AZs，因此要將第 3 個 private subnet 刪掉
> location: devops-manager-master/ansible/camelot/roles/infrastructure/tasks/vpc.yml

> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/vpc_yml_1.jpg) 

> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/vpc_yml_2.jpg) 

### 建立 us-west-1 的 inventory

如果有該 region 的 inventory 則不需要新建，直接修改即可
> location: devops-manager-master/ansible/camelot/inventories
* 建立一個資料：dev-us-west-1 
> `$ mkdir dev-us-west-1`

* 將 dev-us-east-1 下的 'host' 跟 'group_vars/' 複製到 dev-us-west-1 資料夾下
> `$ cp -Rf ./dev-us-east-1/* ./dev-us-west-1`  
> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/inventory_us-west-1.jpg)

* 修改 all.yml
> 填上要使用的 region, AZs 與 ec2_ssh_key 
> ec2_ssh_key 指的是 AWS Console --> EC2 --> Key Pairs 裡 key 的名稱  
> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/all_yml.jpg)


* 將 key pairs import 到 AWS 上
> 在 us-west-1 上沒有 zyxel-camelot 這把 key  
> 1. 先到 us-east-1 的 parameter store 將 `zyxel-camelot-dev-us-east-1.pem` 儲存到 local。檔名為 `zyxel-camelot-dev-us-west-1.pem`   
> 2. 也要存到 Ubuntu 的 ~/.ssh/，並且權限設定為 400   
> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/parameter_store_01_yml.jpg)  
> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/parameter_store_02_yml.jpg)  

> 3. 將 `zyxel-camelot-dev-us-west-1.pem` import 到 AWS Console --> EC2 --> Key Pairs 裡(Region: us-west-1, Key Name: zyxel-camelot)  
> import 方式請參考：[How can I use a single SSH key pair for all my AWS Regions?](https://aws.amazon.com/tw/premiumsupport/knowledge-center/ec2-ssh-key-pair-regions/)  



* 修改 infrastructure.yml
> 練習時都將 type 改成 t3.small
> vpn subnet 要與高荃確認已存在有的 private IP，不能與現有的重複  
> us-west-1 region 只有兩個 AZs，因此要將第 3 個 private subnet 刪掉  
> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/infrastructure_yml.jpg)


### 修改 info.js
> us-west-1 region 只有兩個 AZs，因此要將第 3 個 private subnet 刪掉  
> 將這個刪掉 `private_subnet_three_subnet_id: {{private_subnet_three_subnet_id_fact}}`  
> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/info_j2.jpg)


### 執行 make infra 的指令去自動建立 virtual envirnment 並且安裝需要的軟體

* Makefile 有點像 shell script，裡面已經有定義好的 function 
* 到 Makefile 所在的目錄下執行 make 指令
* 使用 make infra 去自動
> 1. 建立 virtual envirnment 
> 2. 安裝需要的軟體  
> 3. 執行 provision_infra.yml
> 結果：執行後一定會出現錯誤  
>> 會出錯的主要原因是在 Makefile 裡面並沒有撰寫 activate venv 的動作  
>> 一般我們在部屬的時候會在 EC2 上執行，並且有一個 crontab 去執行啟用 venv 的動作，因此並沒有將這一塊寫在 Makefile 中

```
$ cd /devops-manager-master/ansible/camelot
$ make infra
```

### 啟用 virtual environment
``` $ source ./venv/bin/activate```

### 使用 Windows 內建 VPN 服務建立 L2TP 連線

> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/VPN_setting.jpg)

> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/VPN_connect.jpg)


### 建立部屬用的環境 -- 執行 provision_infra.yml 

```$ ansible-playbook provision_infra.yml -i inventories/${CAMELOT_ENV}-${CAMELOT_REGION}/ -vvvv --key-file ${CAMELOT_KEY_PATH}```  

> ```$ ansible-playbook provision_infra.yml -i inventories/dev-us-west-1/ -vvvv --key-file ~/.ssh/zyxel-camelot-dev-us-west-1.pem```


### 手動建立環境的部分(只需要設定一次)
https://redmine.marketplace.zyxel.com/projects/zyxel-camelot/wiki/Checklist_for_Deploying_a_New_Infra

> ### step 1. create an ACM for Camelot portal

>> * 到 AWS Console --> AWS Certificate Manager 去建立一個新的 Certificate
>>> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/ACM_00.jpg)   
>>> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/ACM_01.jpg)  

>> * 剛 Request 的時候需要一點作業時間才會建立好
>>> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/ACM_02.jpg)  
>>> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/ACM_03.jpg)  

> ### step 2. copy base AMI for us-east-1

>> * 尋找要 copy 的 AMI ID
>>> 在 us-east-1 尋找 `zyxel-camelot-base` 開頭的 AMI
>>> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/copy_ami_00.jpg)  

>> * 使用 AWS CLI 執行 copy  
>>> 我將名稱取為：zyxel-camelot-base-image-ubuntu-20211202-Ada-created  

>>> ```$ aws ec2 copy-image --source-image-id ami-xxxxxxxxxxxxx --source-region us-east-1 --region us-west-1 --name "zyxel-camelot-base-image-ubuntu-20211202-Ada-created" --profile zyxel-camelot-dev```  

>>> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/copy_ami_01.jpg)  

>> * 如果沒有此步驟會出現以下錯誤  
>>> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/error_empty_ami.jpg) 

### 部屬 apps 
```$ make apps```

> 目前部屬會無法部屬完成，只有 us-east-1 有 credential file   
> 後續會練習手動做以下兩個
> 1. KMS 加解密  
> 2. [CloudFront 設定 CDN 並且將 policy 推至 S3。只能透過 CDN 存取 S3 Object](https://github.com/Ada-Chen2531/Document/blob/main/%5BAnsible%5DDeploy_Camelot_Frontend.md)  

> ![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/error_credential.jpg) 
