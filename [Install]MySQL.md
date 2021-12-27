### 安裝環境
* OS: Ubuntu 20.04

### 安裝 MySQL
`$ sudo apt install mysql-server`

### MySQL 初始權限設定
`$ sudo mysql_secure_installation`

> 在 Windows 10 會使用 HeidiSQL 去存取 Ubuntu 的 MySQL，因此要開通遠端登入    
>> `Disallow root login remotely? (Press y|Y for Yes, any other key for No) :` --> 輸入任意鍵去 skip  

> 按照文件中的介紹去設定  
> [MYSQL設定Root帳號密碼與初始權限(Ubuntu 20.04)](https://www.albert-yu.com/blog/mysql%E8%A8%AD%E5%AE%9Aroot%E5%B8%B3%E8%99%9F%E5%AF%86%E7%A2%BC%E8%88%87%E5%88%9D%E5%A7%8B%E6%AC%8A%E9%99%90ubuntu-20-04/)

### 讓 MySQL 監聽其它網路介面

* 執行以下指令，可以查看目前 MySQL 所監聽的 port
* 預設不允許其他遠端連線(只有 127.0.0.1 可以連線)
> `$ sudo netstat -tlnp | grep mysqld`

* 以 Ubuntu Server 來說，調整 MySQL 監聽的網路介面的設定檔路徑是 `/etc/mysql/mysql.conf.d/mysqld.cnf` 

* 綁定所有網路介面卡，設定成 0.0.0.0

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/mysql_setting_01.jpg)

* 設定完後重新執行 MySQL 的服務，新設定就會生效
> `$ sudo systemctl restart mysql`

* 此時再檢查一次監聽的 port
> `$ sudo netstat -tlnp | grep mysqld`

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/mysql_setting_02.jpg)

* 參考資料：
> [如何連線到遠端的Linux + MySQL伺服器？](https://magiclen.org/mysql-remote/)

### 新建一個 root remote user
* 先進入 MySQL
> `$ mysql -u root -p`

* 建立一個可以從任意主機都能登入的 root (% 表示任意主機)
> `mysql> $ CREATE USER 'user'@'localhost' IDENTIFIED BY 'PASSWORD';`  
> ex: `$ CREATE USER 'root'@'%' IDENTIFIED BY 'PASSWORD';`

### 給新建的 root remote user Grant 權限
* 你也可以指定使用者存取伺服器上所有資料庫的權限，只需將資料庫名稱替換成 * 即可：
> `mysql > $ GRANT ALL PRIVILEGES ON database_name.* TO 'user'@'localhost';`  
> ex: `$ GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';`

* 參考資料：
> [[MySQL] 如何開啟遠端連線的權限，允許遠端裝置連線資料庫](https://note.charlestw.com/remote-connection-to-mysql/)

### 參考文章
> [Ubuntu 20.04 LTS 安裝 MySQL Server](https://www.ltsplus.com/linux/ubuntu-20-04-lts-install-mysql-server)

