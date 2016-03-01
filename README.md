##SIP簡介

由於路由式 IP 網路的普及化，企業與個人都希望降低電信費用，我們何不利用 IP 網路的數據傳輸來實作語音服務呢？本篇是筆者研讀許多 VoIP 相關文件後所整理的筆記，透過 SIP 不僅可以實現通話及會議通訊功能，也同時擁有視訊會議與訊息傳輸的技術，不過你可能會想，若只局限於 PC to PC 間的通話，是否對於使用者來說方便性過於狹隘？但是對於這部份，筆者最後也整理了幾個 APP client 提供使用者作為參考，讓你的智慧型手機(android、iOS)也可以享有與 PC 相同的服務品質。

## SIP 請求(Requests)與回應(Responses)
### SIP 請求
SIP 的六項基本的方法向伺服器發出請求，以下簡單敘述：

| 方法     | 敘述  | 
| :-----: | :------------------ | 
| INVITE  | 建立通話請求          |
| ACK     | 確認 INVITE 完成     |
| OPTIONS | 查詢狀態與其他相關資訊 | 
| BYE     | 終止通話             |
| CANCEL  | 取消正在請求的通話     |
| REGISTER| 註冊使用者            |

### SIP 回應
SIP 回應分成 Provisional (暫時)及 Final (決定)兩類，Provisional 為 1XX ， Final 則包含 1XX ~ 6XX 。

| 代碼  | 敘述                                 | 
| :--: | :----------------------------------- | 
| 100  | 暫時回應訊息，表示收到請求，繼續處理後續請求|
| 180  | 被呼叫方收到 INVITE 請求，此時電話生想起  |
| 200  | 通話連線成功                           |
| 400  | 請求錯誤                              |
| 401  | 未授權，要求使用者身分驗證               |
| 404  | 使用者不存在於指定 domain               |
| 408  | 請求超時                              |
| 486  | 忙碌                                 |
| 5XX  | 伺服器錯誤                            |
| 6XX  | 全域錯誤                              |


## SIP 安裝教學

#### 作業系統與 SIP 套件：

*   Ubuntu - 14.04 Server
*   OpenSIPS - 2.1 

#### 更新 apt-get：

```
sudo apt-get update
```

#### 下載安裝相關套件：

```
sudo apt-get install git make bison flex & 
mysql-server mysql-client libmysqlclient-dev &
libncurses5 libncurses5-dev 
```

#### 下載 OpenSIPS 套件:

```
cd ~/
sudo git clone https://github.com/OpenSIPS/opensips.git -b 2.1 opensips_2_1
```

#### 修改配置檔：

```
sudo nano ~/opensips_2_1/Makefile.conf.tmplate
```
> 移除 exclude_modules 中 db\_mysql 並儲存

#### 安裝OpenSIPS：

```
cd ~/opensips_2_1
sudo make all
sudo make install
```

#### 安裝完畢後，修改部分opensipsctlrc文件，如下：

   
```
sudo nano /usr/local/etc/opensips/opensipsctlrc

修改如下，將部分註解刪除

## your SIP domain
SIP_DOMAIN=ubuntustudio
## chrooted directory
# $CHROOT_DIR="/path/to/chrooted/directory"
## database type: MYSQL, PGSQL, ORACLE, DB_BERKELEY, or DBTEXT, 
## by default none is loaded
# If you want to setup a database with opensipsdbctl, you must at least specify
# this parameter.
DBENGINE=MYSQL
## database host
DBHOST=localhost
## database name (for ORACLE this is TNS name)
DBNAME=opensips
# database path used by dbtext or db_berkeley
DB_PATH="/usr/local/etc/opensips/dbtext"
## database read/write user
DBRWUSER=opensips
## password for database read/write user
DBRWPW="opensipsrw"
## database super user (for ORACLE this is 'scheme-creator' user)
DBROOTUSER="root"
# user name column
USERCOL="username"
```
	
> 如果出現沒有權限修改 opensipsctlrc 請先修改 /usr/local/etc/opensips 權限

#### 執行建立資料庫腳本：
   
```
/usr/local/sbin/opensipsdbctl create

```

> 你會看到如下方結果：
> 
```
MySQL password for root: 
INFO: test server charset
INFO: creating database opensips ...
INFO: Core OpenSIPS tables succesfully created.
Install presence related tables? (y/n): y  
INFO: creating presence tables into opensips ...
INFO: Presence tables succesfully created.
Install tables for imc cpl siptrace domainpolicy carrierroute userblacklist? (y/n): y
INFO: creating extra tables into opensips ...
INFO: Extra tables succesfully created.
```	
> 如果執行出現錯誤，可以從 /var/log/syslog 檢視錯誤資訊。

##### 常見錯誤：
* var/run 路徑權限不足

```
sudo chmod 777 /var
sudo chmod 777 /var/run
```
* opensips.cfg 權限不足

```
sudo chmod 755 /usr/local/etc/opensips/opensips.cfg
```

#### 設定 OpenSIPs listen：
使用你慣用的編輯軟體如: vim ，開啟 /usr/local/etc/opensips/opensips.fcg 進行設定。

``` 
...
listen=udp:10.21.20.111:5060
...
```

#### 新增 Domain 至資料庫中：
首先進入 MySQL ，並且輸入下列指令建立 Domain 至 opensips.domain 資料表。

```
INSERT INTO opensips.domain(domain) VALUES('10.21.20.111');
```

並且可以透過搜尋語法，確認是否有將該筆資料新增進資料表中。

```
SELECT * FROM opensips.domain;
```

最後，離開 MySQL。

```
exit;
```

#### OpenSIPS 操作：
啟動：

```
/usr/local/sbin/opensipsctl start
```
	
暫停：

```
/usr/local/sbin/opensipsctl stop
```
	
重新啟動：

```
/usr/local/sbin/opensipsctl restart
```
	
> 如果執行時出現錯誤，請檢查 opensips.cfg 、 /var 與 /var/run 權限是否有誤。

## 測試軟體

* Mac OS : [Yate Client](http://yateclient.yate.ro/)、Zoiper
* Android : ECOA Sip (可至 Play 商店下載)
* iOS :  Zoiper (可至 App Store 下載)

## 參考
1. [登入 error code 483 處理方式](http://www.wjxfpf.com/2015/10/331846.html)
2. [open source VOIP Software](http://www.voip-info.org/wiki/view/Open+Source+VOIP+Software)



