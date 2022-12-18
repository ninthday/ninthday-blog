---
title: "Laravel CICD with Github Actions"
author: "Tuvix Shih"
slug: "laravel-cicd-github-actions"
featured_image: "laravel.png"
cover: "/images/github.png"
tags: ["laravel", "github"]
date: 2021-02-28T15:57:46+08:00
draft: false
---

GitHub Actions 為 2019 年 11 月 GitHub 推出的服務，可以用來測試、封裝、發佈或是部署程式碼，一連串的動作建立自動化的流程，達到持續整合/持續部署（CI/CD）的目的。

Github 讓開發者能將一些重複的動作寫成腳本，使其他開發者也可以引用這些 Action，把這些 Action 組合起來執行，就能夠完成前述 CI/CD 的過程。GitHub 為此成立了一個 Action 的 [Marketplace](https://github.com/marketplace?type=actions)，可以搜尋到其他人上傳的 Action。

這篇文章中，將使用 GitHub Actions 將 Laravel 專案測試及部署至自己的 Linux 伺服器上。

## 1. GitHub Actions 名詞
- Workflow（工作流程）：可設定為一個自動化流程的程序，Workflow 由一個或多個 Job 組成，可經由事件觸發執行，是 YAML 格式檔案放在 `.github/workflows` 資料夾中。
- Job（任務）：每個 Job 由多個 Step 組成，一個 Job 會在一個新的執行實例中（instance）執行。不同的 Job 可以同時執行，也可以依照前面的 Job 狀態依順序執行。
- Step（步驟）：每個 Step 都是在相同的執行實例中執行，Step 可以是執行指令或是 Action。每個 Step 可以執行一個或多個 Action。
- Action（動作）：Workflow 最小單位。

## 2. Workflow 文件
GitHub Actions 的 workflow 文件為 YAML 格式，文件名稱可以自定但副檔名為 `.yml`。GitHub 只要在 `.github/workflows` 資料夾中發現有 `.yml` 文件就會自動執行。

這個 YAML 文件的語法分為幾個部分
- **name**: workflow 名稱
- **on**
	指定觸發 workflow 的條件（事件）
- **jobs**
	workflow YAML 文件的主要部分，`jobs` 中包含一或多個 `job`，列出每一個 `job_id`，可以使用 `needs` 關鍵字指定和其他 `job` 的依賴關係。使用 `runs-on` 來指定執行的實例
	```yaml
	jobs:
	  my_first_job:
	    name: My first job
	    runs-on: ubuntu-20.04
	  my_second_job:
	    name: My second job
	    needs: my_first_job
	```
	> 上面例子有兩個 `job`，`job_id` 分別為 `my_first_job` 和 `my_second_job`，`my_second_job` 依賴於 `my_first_job`，`my_second_job` 需要等待 `my_second_job` 完成才能執行。
- **step**
	每個 `job` 下有多個 `step`，每個 `step` 主要有三個部分 ，
	- `jobs.<job_id>.steps.name`:  `step` 名稱
	- `jobs.<job_id>.steps.run`: `step`執行的指令或是 action
	- `jobs.<job_id>.steps.env`: `step` 執行時的環境變數

綜合以上的基本結構，一個基本的 workflow YAML 結構如下：
```yaml
name: Greet Everyone
on: push
jobs:
  my-job:
    name: My Job
    runs-on: ubuntu-latest
    steps:
    - name: Print a greeting
      env:
        MY_VAR: Hi there! My name is
        FIRST_NAME: Mona
        MIDDLE_NAME: The
        LAST_NAME: Octocat
      run: |
        echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
```

## 3. 建立 Workflow
在 GitHub Actions 的 Workflow 的設計上，我會趨向把持續整合（CI）及持續部署（CD）分為兩個獨立的 Workflow YAML  檔，在每次整合是都能先經過測試程序，在程式碼較為穩定的時候再部署至指定的伺服器。

### 3.1 持續整合（CI: Continuous Integration）
軟體開發的過程中，測試是相當重要的一環。以目前軟體開發逐漸提高的複雜度，已經不容易使用傳統長時間的測試方式，需要更即時的測試。加上軟體開發大都以團隊協同合作進行，在彼此程式碼整合前需要先經過持續的測試。

在 `.github/workflows` 資料夾中建立 `laravel-ci.yml` 內容如後。
```yaml
name: Laravel 6 Continuous Integration 
on:
  push:
    branches:
      - master
jobs:
  laravel-tests:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
      - name: Setup PHP version
        uses: shivammathur/setup-php@v2
        with:
          php-version: "7.4"
      - name: Copy ENV Laravel Configuration for CI
        run: |
          php -r "file_exists('.env') || copy('.env.ci', '.env');"
      - name: Install Dependencies (PHP vendors)
        run: composer install -q --no-ansi --no-interaction --no-scripts --no-suggest --no-progress --prefer-dist
      - name: Generate key
        run: php artisan key:generate
      - name: Directory Permissions
        run: chmod -R 777 storage bootstrap/cache
      - name: Create DB and schemas
        run: |
          mkdir -p database
          touch database/database.sqlite
          php artisan migrate
      - name: Run PHPCS Coding Standards
        run: |
          vendor/bin/phpcs --colors
      - name: Execute tests (Unit and Feature tests) via PHPUnit
        env:
          DB_CONNECTION: sqlite
          DB_DATABASE: database/database.sqlite
        run: vendor/bin/phpunit --testdox
```

### 3.2 持續部署（CD: Continuous Deployment）
在前段的說明中有提到在程式碼穩定之後再部署，「穩定」這裡我的定義會是在 PR 確認 merge 完成關閉時進行。主要是透過 SSH 的方式，連結至指定伺服器執行部署的腳本 (shell script)。

#### 3.2.1 準備工作：
1. 設定部署帳號
	由於是使用 SSH  方式讓 GitHub Action 可以訪問我們的伺服器部署，在目標伺服器上開啟一個部署專用的帳號，限制帳號的權限不要太大提高安全性。
2. 建立伺服器登入憑證
    在目標伺服器上產生使用 `ssh-keygen` 指令產生公私鑰，將公鑰加入 `authorized_keys`，私鑰加入 GitHub 的加密變數中，做為連線登入使用的憑證。
    ```bash
    $ cd ~/.ssh/
    $ ssh-keygen -f cicd-github
    $ cat ~/.ssh/cicd-github.pub >> ~/.ssh/authorized_keys
    ```
3. 修改 sudoer 權限
    因為我使用 `nginx` 作為 Web Server，`nginx` 本身無法處理 PHP 程式，需要透過 PHP-FPM (FastCGI Process Manager) 來解譯處理 ，部署的最後會需要重新 Reload PHP-FPM Service，`systemctl reload` 為 root 才能執行的權限，修改 `sudoer` 讓 `cicd-github` 可以執行 PHP-FPM Service Reload。
    ```bash
    $ sudo visudo
    cicd-github  ALL=NOPASSWD: /bin/systemctl reload php7.4-fpm.service
    ```
    > 於 GitHub Actions 執行時無法於 sudo 過程中輸入密碼，在這裡設定為 `NOPASSWD`，同時也限制帳號只能執行 reload 這個指令
    
4. GitHub 設定加密變數
    GitHub Actions 執行的過程中，不會中斷執行等待輸入，因此設定加密變數 (Secrets) 讓 Actions 執行期間帶入需要的變數內容。
	加入加密變數的方法：GitHub Repo --> Settings --> Secrets --> New repository secret
    主要需要的參數有 `SSH_HOST`、`SSH_USERNAME`、`SSH_PRIVATE_KEY`
    
    - SSH_HOST：伺服器位址
    - SSH_USERNAME：登入伺服器的帳號名稱
    - SSH_PRIVATE_KEY：登入的私鑰，由伺服器 `cat ~/.ssh/cicd-github` 取得前面第2步驟的私鑰內容
    > 若 SSH 使用的不是預設的 Port，可以增加 `SSH_PORT` 變數存入 Port 號碼

5. 新增 Personal access token
    在部署腳本中主要使用 Git 的方式，來拉取程式碼至伺服器上，對於私人儲存庫 (private repositories) 執行期間將要求必須輸入 GitHub 帳號密碼，將會造成執行中斷。我們可以透過建立  GitHub **Personal access tokens**，讓 Token 代表 GitHub Account 可以存取 GitHub API，同時也可以增加安全性，當懷疑 Token 外洩時，可以重新產生新的 Token 棄用舊的。
    建立 Personal access token方法：[GitHub Personal access token](https://github.com/settings/tokens) --> Generate new token
    
6. Git Clone Repository
    在目標伺服器上，以部署用帳號 (cicd-github) 使用 https 方式 Clone Repository 至本地。
    ```bash
    $ git clone https://<Github Account>:<Personal access token>@github.com/<Repository>.git
    ```
#### 3.2.2 伺服器部署腳本
建立一個 Shell Script ，並將檔案名稱設定為 `server_deploy.sh`，腳本內容如後。
```bash
#!/bin/sh
set -e

echo "Deploying application ..."

# Enter maintenance mode
(php artisan down --message 'The app is being (quickly!) updated. Please try again in a minute.') || true
    # Update codebase
    git checkout deploy
    git fetch origin deploy
    git reset --hard origin/deploy

    # Install dependencies based on lock file
    composer install --no-interaction --prefer-dist --optimize-autoloader

    # Migrate database
    php artisan migrate --force

    # Clear cache
    php artisan cache:clear
    php artisan route:clear
    php artisan config:clear
    php artisan view:clear

    # Update database classmap
    composer dump-autoload

    # Optimizing Configuration Loading
    php artisan config:cache

    # Reload PHP to update opcache
    sudo systemctl reload php7.4-fpm.service
# Exit maintenance mode
php artisan up

echo "Application deployed!"
```
記得讓 `server_deploy.sh` 檔案有執行權限才能部署。
```bash
$ chmod u+x server_deploy.sh
```

#### 3.2.3 建立部署 GitHub Action
在 `.github/workflows` 資料夾中建立 `laravel-cd.yml` 檔案。Deployment Action 在 GitHub deploy 分支 PR (pull request) 關閉時觸發，並在確認 `merge` 後才執行部署。 
```yaml
name: Continuous Deployment Laravel 6
on:
  pull_request:
    branches:
      - deploy
    types: [closed]

jobs:
  laravel-deploy:
    runs-on: ubuntu-20.04
    if: github.event.pull_request.merged == true
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Deployment
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_HOST }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          username: ${{ secrets.SSH_USERNAME }}
          script: |
            cd /var/www//html/repository-path
            ./server_deploy.sh
```
> Note:
> 若需要 SSH Action 其他輸入參數可以參考 [SSH for GitHub Actions](https://github.com/appleboy/ssh-action)
---
參考資料：
1. [GitHub Actions 基礎介紹](https://ibe.tw/getting-started-with-github-actions/)
2. [GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)
3. [Build, Test, and Deploy Your Laravel Application With GitHub Actions](https://www.twilio.com/blog/build-test-deploy-laravel-application-github-actions)
4. [How to create a CI/CD for a Laravel application using GitHub Actions](https://blog.logrocket.com/how-to-create-a-ci-cd-for-a-laravel-application-using-github-actions/)
5. [Push deploy a Laravel app for free with GitHub Actions](https://laravel-news.com/push-deploy-with-github-actions)

