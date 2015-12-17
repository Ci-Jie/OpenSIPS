**安裝OpenSIPS** 
=======
#####更新日期：2015/11/23
### **版本**
*   Ubuntu - 14.04 Server
*   OpenSIPS - 2.1 

### **安裝**
1. **更新 apt-get：**

   ``sudo apt-get update ``

2. **下載安裝相關套件：**

  ``sudo apt-get install git ``
  ``sudo apt-get install make ``
  ``sudo apt-get install bison ``
  ``sudo apt-get install flex``
  ``sudo apt-get install mysql-server mysql-client``
  ``sudo apt-get install libmysqlclient-dev``
  ``sudo apt-get install libncurses5 libncurses5-dev``

3. **下載OpenSIPS套件：**

    ``cd ~/``<br>
    ``sudo git clone https://github.com/OpenSIPS/opensips.git -b 2.1 opensips_2_1``

4. **修改配置檔：**

   ``sudo nano ~/opensips_2_1/Makefile.conf.tmplate``<br>
   ``移除 exclude_modules 中 db_mysql 並儲存``

5. **安裝OpenSIPS：**

   ``cd ~/opensips_2_1``<br>
   ``sudo make all``<br>
   ``sudo make install``

6. **安裝完畢後，修改部分opensipsctlrc文件，如下：**

   ``sudo nano /usr/local/etc/opensips/opensipsctlrc`` 
   
	```
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
	
   >備註：如果出現沒有權限修改opensipsctlrc 請先修改 opensips 路徑權限
   
   ``sudo chmod 755 /usr/local/etc/opensips``

7. **執行建立資料庫腳本：**

   ``/usr/local/sbin/opensipsdbctl create``
   
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
	
   >備註：如果執行出現錯誤，可以從 syslog 檢查錯誤
   
	>``sudo nano /var/log/syslog``
	
   >常見錯誤：
   
	>* var/run 路徑權限不足
	
	>	``sudo chmod 777 /var``<br>
	>	``sudo chmod 777 /var/run``
	
	>* opensips.cfg 權限不足
   
	>	``sudo chmod 755 /usr/local/etc/opensips/opensips.cfg``

8. **設定OpenSIPs listen：**

	``sudo nano /usr/local/etc/opensips/opensips.fcg``
	
	找到 listen 並修改成 Server IP (10.21.20.111為作者範例，請依照自己 Server IP 修改)
	
	``listen=udp:10.21.20.111:5060 #CUSTOMIZE ME``

9. **新增 Domain 至資料庫中：**

	先進入 MySQL 資料庫中
	
	``mysql -u root -p``
	
	使用 SQL language 新增 Domain 至 opensips.domain 資料表

	``INSERT INTO opensips.domain(domain) VALUES('10.21.20.111');``
	
	可以透過 Select 語法查詢是否正確加入資料表中
	
	``SELECT * FROM opensips.domain;``

	離開 MySQL
	
	``exit;``

10. **啟動OpenSIPS：**

	啟動：``/usr/local/sbin/opensipsctl start``
	
	暫停：``/usr/local/sbin/opensipsctl stop``
	
	重新啟動：``/usr/local/sbin/opensipsctl restart``
	
	>備註：如果執行出現錯誤，請檢查 opensips.cfg 、 /var 與 /var/run 權限

### **參考**
1. http://www.verydemo.com/demo_c230_i1005.html
2. http://www.wjxfpf.com/2015/10/331846.html



