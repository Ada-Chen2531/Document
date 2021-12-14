### 此文件目標
CloudFront CDN 是由 Amazon 提供的一套覆蓋全球的 CDN 網絡  
S3 會放 CDN 用的圖檔，然後權限是只給 CloudFront

目標：透過 CloudFront 建立 CDN 的同時，將存取的 policy 自動推送至 S3

### create distribution
![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/CloudFront_00.jpg)


### 選擇要設定的 S3 bucket
![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/CloudFront_01.jpg)

### 新建一個 OAI，並設定只有 CloudFront 可以存取

* 選擇將 policy 推送到 S3 bucket

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/CloudFront_02.jpg)

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/CloudFront_03.jpg)

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/CloudFront_04.jpg)

### CloudFront 建立完後到 S3 檢查
* AWS Console --> Buckets --> 選擇設定的 bucket --> Permissions  
* 會看到 policy 自動設定到 S3

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/CloudFront_05.jpg)

### S3 上放置一些檔案

* index.html 裡面寫一行 HTML

`<h1>This is an S3</h1>`

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/CloudFront_06.jpg)


### 設定 S3 Static Website Hosting

* AWS Console --> Buckets --> 選擇設定的 bucket --> Properties --> Static website hosting --> click "Edit" button

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/CloudFront_07.jpg)

### 直接透過該 website endpoint 存取 index.html

* 結果會被拒絕

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/CloudFront_08.jpg)


### 將 website endpoint 改成 CDN domain 去存取 index.html

* 結果是可以存取成功的

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/CloudFront_09.jpg)
