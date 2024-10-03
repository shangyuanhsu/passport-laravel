# Laravel 11 + Laravel Passport + Sail

This guide will walk you through setting up a Laravel 11 project using Laravel Passport for API authentication and Sail for containerized development with Docker.

## Create Project

Navigate to the directory where you want to create the project and run the following command:

```bash
laravel new passport-laravel
```
During project setup, you’ll be prompted with some choices. Here’s a reference for selecting options:

```bash

Would you like to install a starter kit?
> laravel Breeze

Which Breeze stack would you like to install?
> Blade with Alpine

Would you like dark mode support? 
> no

Which testing framework do you prefer?
> Pest

Would you like to initialize a Git repository?
> yes

Which database will your application use?
> MySQL

Default database updated. Would you like to run the default database migr… 
> no ## We'll migrate after setting up the database.
```

## Setting Up Environment & Sail

###Install Sail
Laravel 11 comes with Sail by default. If it's not installed, you can add it using:

```bash
composer require laravel/sail --dev
```
Sail Installation
Run the following command to configure Sail:

```bash
php artisan sail:install
```
Choose MySQL as the database service:

```bash
Which services would you like to install?
> MySQL
```

Customize docker-compose.yml
You can adjust ports in the docker-compose.yml if you have multiple services running:

```yaml
ports:
    - '${APP_PORT:-80}:80'
    - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'

# Custom port example:
    - '${APP_PORT:-8889}:80'
    - '${VITE_PORT:-8000}:${VITE_PORT:-5173}'

mysql:
    ports:
    - '${FORWARD_DB_PORT:-3306}:3306'

# Custom port example:
    - '${FORWARD_DB_PORT:-8888}:3306'
```

In your .env file, add the following variables to manage file permissions in the Docker container:

```bash
WWWUSER=1000
WWWGROUP=1000
```
These variables ensure that the Docker container’s web server (e.g., Nginx) has proper read/write access to files.

### Start Docker
Install Docker Desktop: Docker Desktop Installation
Ensure Docker Desktop is running, then in your project directory, run:

```bash
docker-compose -f docker-compose.yml up
```
This will create a container visible in Docker Desktop, and the project will be accessible on the port specified in docker-compose.yml (e.g., 80:80). You may encounter an error at this stage, but we will fix it in the next steps.

### Set Up MySQL
Use MySQL Workbench or any other tool to create your database. Credentials are in your .env file (username is root, and password defaults to password). The database name will match your project.

After creating the database, open the Sail container by navigating to Docker Desktop, select the container, and run the following command:

```bash
php artisan migrate
```
This will create the necessary tables, and the project should now load without errors.

## Install Passport

In the Sail container, run the following command to install Passport:

```bash
php artisan install:api --passport
```
You will be prompted with:

```bash
> Would you like to use UUIDs for all client IDs? (yes/no) [no]:
yes

> API scaffolding installed. Please add the [Laravel\Passport\HasApiTokens] trait to your User model.
Update User Model
```

Go to app/Models/User.php and add the Passport trait:

```php
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasFactory, Notifiable, HasApiTokens;

    ...
}
```
Update auth.php
In config/auth.php, update the API guard to use Passport:

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
 
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

## Create OAuth Client

To register an external application for OAuth, run the following in the Sail container:

```bash
php artisan passport:client
```
You'll be prompted:

```bash
Which user ID should the client be assigned to? (Optional):
> 

What should we name the client?:
> client001 

Where should we redirect the request after authorization? [http://localhost/auth/callback]:
> http://localhost:8889/auth/callback

INFO  New client created successfully.  

Client ID .................. 9d2783c6-6a3a-4980-a52a-01b07f00d8fb  
Client secret .................. 5PalPKU5YN32nT98w5zvELYtgM722jCyOKP5GOxO
```

## Authorize OAuth Client

To authorize a client, make a GET request like this:

```bash
localhost/oauth/authorize?
client_id={$client_id}&
redirect_uri={$redirect_uri}&
response_type=code
```

Example:

```bash
localhost/oauth/authorize?client_id=9d2783c6-6a3a-4980-a52a-01b07f00d8fb&response_type=code&redirect_uri=http://localhost:8889/auth/callback
```

This will prompt for login, and after granting authorization, you will receive an authorization code in the redirect URL, such as:

```bash
http://localhost:8889/auth/callback?code=def502007f5146ba...
```

## Get Access Token

Use the authorization code to get an access token via a POST request:

```bash
/oauth/token

grant_type:authorization_code
client_id:{$client_id}
client_secret:{$client_secret}
redirect_uri:{$redirect_uri}
code:{$code}
```

## Fetch User Data Using Access Token

To fetch user details, send a GET request to:

```bash
/api/user

// Authorization: Bearer {access_token}

```

---

# laravel 11 + laravel passport + sail

---

## create project

到要安裝專案的地方下以下指令
```powershell
laravel new passport-laravel
```

會要你選一些專案的設定
以下是選擇參考

```powershell
Would you like to install a starter kit?
> laravel Breeze

Which Breeze stack would you like to install?
> Blade with Alpine

Would you like dark mode support? 
> no

Which testing framework do you prefer?
> Pest

Would you like to initialize a Git repository?
> yes

Which database will your application use?
> MySQL

Default database updated. Would you like to run the default database migr… 
> no ## 因為我們資料庫還好有好，等一下再migrate

```

---

## 環境＋開啟專案

### 安裝sail

現在laravel11預設都會安裝sail
如果沒有就下以下指令進行安裝
```powershell
composer require laravel/sail --dev
```

啟用 sail 
以下指令會產生 docker 需要的docker-compose.yml
```powershell
php artisan sail:install
```

```powershell
Which services would you like to install?
> MySQL
```

在docker-compose.yml有幾個地方可以調整
假設有多個服務你可以將port做調整與分配

```powershell
ports:
    - '${APP_PORT:-80}:80'
    - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'

    
    這邊可以設定成別的
    - '${APP_PORT:-8889}:80'
    - '${VITE_PORT:-8000}:${VITE_PORT:-5173}'
    
.... 
    
 mysql:
    ports:
    - '${FORWARD_DB_PORT:-3306}:3306'
        
    這邊可以設定成別的
    - '${FORWARD_DB_PORT:-8888}:3306'
```

然後.env需要新增
```powershell
WWWUSER=1000
WWWGROUP=1000
```

文件中設置 `WWWUSER` 和 `WWWGROUP` 變量主要與應用的文件權限有關，確保Docker容器內的Web服務器（例如 Nginx）能夠正確地讀取和寫入文件。

- `WWWUSER`：指的是運行容器內 Web 服務器的用戶 ID。
- `WWWGROUP`：指的是運行容器內 Web 服務器的用戶組 ID。


### 啟動docker

這之前可安裝Docker desktop
https://www.docker.com/products/docker-desktop/

確定安裝完先幫忙開啟到畫面中
回到專案後

下以下指令來創建Container
```powershell
docker-compose -f docker-compose.yml up     
```
這時候就會看到Docker desktop會多一個跟你專案一樣名字的Container

都安裝完成後選擇 80:80(依照你設定的port) 就可以看到專案畫面了
不過畫面應該會報錯誤，那我們要緊接著往下做

---

### 處理Mysql

我自己是使用MySQL Workbench，不管怎麼樣就是去建資料庫就對了
username就是env設定的
密碼預設是password
然後就會看到跟專案一樣名字的DB名稱
接著我們要回到docker進到sail的Container
再選擇到Exec中
去下創建專案預設的table的指令
```powershell
php artisan migrate
```
回到頁面就會正確了！

---

## 安裝passport

回到docker進到sail的Container到Exec中下指令
```powershell
php artisan install:api --passport
```
```powershell
> Would you like to use UUIDs for all client IDs? (yes/no) [no]:
yes

> API scaffolding installed. Please add the [Laravel\Passport\HasApiTokens] trait to your User model.
```

安裝之後，要去程式中加東西
到 app/Models/User.php
我們要use剛剛安裝的東西

```php
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasFactory, Notifiable, HasApiTokens;

...
```

到 config/auth.php
加上api那段
```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
 
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

---

建立用戶/user
畫面上有register

---

## 建立oauth client

申請要用我們系統登入的系統、廠商…
回到docker sail的Exec中下指令
```php
php artisan passport:client
```
```powershell
Which user ID should the client be assigned to? (Optional):
> 

What should we name the client?:
> client001 

Where should we redirect the request after authorization? [http://localhost/auth/callback]:
> http://localhost:8889/auth/callback

   INFO  New client created successfully.  

Client ID .................. 9d2783c6-6a3a-4980-a52a-01b07f00d8fb  
Client secret .................. 5PalPKU5YN32nT98w5zvELYtgM722jCyOKP5GOxO
```

---

## 取得允許oauth

[GET]

```php
localhost/oauth/authorize?
client_id={資料庫passport_laravel.oauth_clients的id}&
redirect_uri={資料庫passport_laravel.oauth_clients的redirect}&
response_type=code

localhost/oauth/authorize?client_id=9d2783c6-6a3a-4980-a52a-01b07f00d8fb&response_type=code&redirect_uri=http://localhost:8889/auth/callback
```

網址這樣過去會需要登入，登入之後會要授權

同意授權之後會回傳code
```
// 大概會長這樣
http://localhost:8889/auth/callback?code=def502007f5146baeb70141c3e934206a17211ec3862b4cf03cbe2fbfda0ab6691a95085c1afed67d0abae206cdf53007b91e9900936f72c6d1787f803cc5369aca04a007a600da56a322e1b746cfee237096064988825eb4b4a1fa2f7de8a5ae2ca1de08b07f8c23351a04804e78a58161ac4b11b41abcab4c4fd5247fc01ad2770d2c6a96cf2b30cb35c04fd938052db845b80211b44df1a1834ea475febb6218a87885620381a0e84f2a5253f6b8ae1dfed5d6375c730436e378d5f519ef76286cdd6c2924df8e5d151ce29da51ee17a80ef0077518d049d359d64dc75ae92d604bcc61df75c0d80a9297a61594a057a8448e009b389cb2c1191c06494c4897d6e31a23e8fc08777288413a9a175eab700ee9de00c88e4ac50e9332119baa96fa949f756fa42109e69467f02cef1fab50fb319a9b9dc90e6f318f7712d1682d228da83fcb7b60d04fc206f7b00fa9947b626fb72f8dc9d511a4464e3d923db2f044b821ca741bdc618632ee365cf7facd
```

---

## 取token

[POST]

```
/oauth/token

grant_type:authorization_code
client_id:{資料庫passport_laravel.oauth_clients的id}
client_secret:{資料庫passport_laravel.oauth_clients的secret}
redirect_uri:{資料庫passport_laravel.oauth_clients的redirect}
code:{取得允許oauth的code}
```

---

## 用token取使用者資料

[GET]

```
/api/user


// Bearer Token
// 放入取token拿回來的access_token
```

---