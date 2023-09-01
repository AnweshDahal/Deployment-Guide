Deployment Setup for JavaScript Web Applications

We will be emulating a server instance using Multipass for Ubuntu, you can do this in any secure sandbox of your liking. This can be Virtual Machines Instances, Cloud Instances, or Physical Machines you have at your access.

## 1. Setting Up a Cloud Instance
> Later

## 2. (For Cloud Instance) Using SSH to connect to a cloud instance using Ubuntu 20.04 LTS
In order to establish a connection to the cloud services you will require a few tools and credentials from the get go these are:
1. SSH program, this comes paired with Ubuntu 
2. Credentials such as *Private Key* and *Public Key*

1. **Create a folder in your home directory and call it `.ssh`**, If the folder already exists, this step can be skipped
2. **Copy the `rsa` key onto the folder**
3. **Make the following addition to the `config` file**, you can create it if it does not exist
```config
Host [host-name] # this can be anything, just make sure it is distinctive
	HostName [IP address of the instance]
	User [username used to connect to the instance]
	IdentityFile [location of the rsa file we copied earlier]
```
4. **Connect to the instance using `ssh`**
```bash
ssh host-name
```
5. **If you might be asked to pass in a password that you have created for the instance.**
## 3. (For Test Instance) Connecting to our Instance Using Multipass
1. **Install Multipass**
```bash
sudo apt install multipass -y
```
2. **Check if Multipass is Installed Correctly**
```bash
multipass -v
```
3. **Install an Ubuntu Instance**
```bash
multipass launch focal --name server # here you can replace focal with the ubuntu version of your choice
```
4. **Check if the instance is running and start it if not**
```bash
multipass list
```

You will see the following output
```Output
Name                    State             IPv4             Image
demo                    Stopped           --               Ubuntu 20.04 LTS
demo-server             Running           10.169.38.66     Ubuntu 20.04 LTS
```
Here you can see that the instance `demo` is `Stopped` and instance `demo-server` is `Running`, if you want to use `demo` just run the following command.
```bash
multipass start demo # replace the name demo with yor insstance name
```
Also, note the IP address as later on we will be using alias to change the name of the server.
5. **Connect to Instance**
```bash
multipass exec bash -- demo # here you can replace the name demo with the one you created earlier
```
After this point the steps you follow to setup the server is the same.

## 4. Setting Up The Server (Pre-Requisites)
1. **Run Update**
```bash
sudo apt update
```
2. **Update `Locale` to use `UTF-8`, I'll be using `vim` but you can use any CLI based text editor, `nano` is shipped by default with Linux**
```bash
sudo apt insall vim # install vim 
# ! IMPORTANT: Use nano if you are not familiar with vim
sudo vim /etc/default/locale # since we are changing files in / folder we need super user access
```
and add the following
```locale
LANG=en_US.UTF-8  
LANGUAGE=en_US.UTF-8  
LC_ALL=en_US.UTF-8
```
3. **Install Nginx**
```bash
sudo apt-get update
sudo apt-get install nginx -y
```
You can check if `nginx` is running by running the following command.
```bash
sudo systemctl status nginx
```
And you want the following response,
![[Pasted image 20230828101512.png]]
You can see that the response is active and (running), you would see this if the service was stopped.
![[Pasted image 20230828101704.png]]
if so you can run the following command to start the service.
```bash
sudo systemctl start nginx
```

> [!WARNING] Important
> If you encounter any other error, you need to consult with nginx docs and support

> [!INFO] Provide an alias to the server
> You can use an alias for the Multipass server by following the steps in this section [[#Aliasing Multipass with an actual domain name]]

4. **Install MySQL Server**
```bash
sudo apt install mysql-server -y
```
5. **Install PHP**
```bash
sudo apt install php-fpm php-cgi php-mysql -y
```
Check if **MySQL** is Running,
```bash
sudo systemctl status mysql
```
You need the following response
![[Pasted image 20230828104834.png]]
6. **Install phpmyadmin**
```bash
sudo apt install phpmyadmin -y
```
And select `apache2`
![[Pasted image 20230828105057.png]]
and select `OK` in next prompt
![[Pasted image 20230828105139.png]]
And `Yes`
![[Pasted image 20230828105155.png]]
Enter a password I have entered `password`
![[Pasted image 20230828105247.png]]
And confirm password
![[Pasted image 20230828105342.png]]
7. **Create a symlink of phpmyadmin**
```bash
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
```
8. **Make the following addition to the Nginx sites-available file** using
```bash
sudo vim /etc/nginx/sites-available
```
Now search for the line that says
```default
index index.html index.htm index.nginx-debian.html;
```
and add the entry `index.php` so the line should look something like this
![[Pasted image 20230828132116.png]]
finally add the following lines below `location \` block
```default
location ~ \.php$ {
	include snippets/fastcgi-php.conf;
	fastcgi_pass unix:/var/run/php/php7.4-fpm.sock
}
```

Reload Nginx to apply changes
```bash
sudo systemctl reload nginx
```

> [!WARNING]
>  Make sure that you update the Unix socket and not TCP socket

now you need to check if **phpmyadmin** is accessible, which can be done by accessing phpmyadmin.

```url
http://test.demoserver.com/phpmyadmin
```
and you want to see this screen
![[Pasted image 20230828133414.png]]

however we have not created a user for phpmyadmin
9. **Create a user for phpmyadmin**
```bash
sudo mysql --user=root mysql
```

```mysql
CREATE USER 'superadmin'@'localhost' IDENTIFIED BY 'password'; # use a stronger password
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION; # you can remove WITH GRANT OPTION if you dont want the user to grant privileges to other user
FLUSH PRIVILEGES;
exit;
```
10. **Create a swap file**
Run this to check if one exists
```bash
free
```
if not
```bash
cd /var
sudo touch swap.img
sudo chmod 600 swap.img
sudo dd if=/dev/zero of=/var/swap.img bs=1024k count=1000
sudo mkswap /var/swap.img
sudo swapon /var/swap.img
```
This concludes the preliminary setup of the server, now we need to install some additional dev tools
## 5. Additional Installations
1. **Install Git**
```bash
sudo apt install git
```
2. **Install Nodejs**
```shell
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash - # replace setup_16.x with the rquired node vrsion
sudo apt install -y nodejs
```
## 6. Deploy application front-end
Now that we have setup our machine we need to deploy the app we worked on.
1. **Clone the repository**
```shell
git clone https://github.com/username/repo-name
```

> [!INFO]
> We will setup CI/CD using GitHub Actions later.

2. **Install the dependencies and build the project**
```bash
cd repo-name
touch .env 
sudo vim .env # paste the environment variables inside
npm install # assuming this is a nodejs application
npm build # the build command of your application found in package.json
```

> [!WARNING]
> As of Ubuntu 20.04, the build files need to be inside `/var/www/html` folder in order to be accessed publicly

3. **Copy built files to `/var/www/html`**
```bash
cp dist/. /var/www/html
```

4. **Update config to SPAs with Routers made using JS Frameworks**
```bash
sudo vim /etc/nginx/sites-available/default
```

Under `location /` make the following change
```default
location / {
# Location Contents
	try_files $uri $uri/ index.html =404;
# Location Contents
}
```
This makes sure that if you request a specific page in your SPA, it won't throw a `404 Not Found`.
5. **Reload Nginx**
```bash
sudo systemctl reload nginx
```

## 7. Deploying Application API
1. **Clone and setup project**
```bash
git clone https://github.com/username/repo-name
cd repo-name
touch .env
vim .env # copy environment variables
npm i
```
2. **Install pm2 to run the server, and start it**
```bash
sudo npm i -g pm2
pm2 start app.js # app.js must be replaced with the path of your api's entry point 
```

## 8. Nginx Reverse Proxy Setup For API
1. **Nagivate to `/etc/nginx/sites-available` and copy the `default` file and rename it to the name of your site**
```bash
cd /etc/nginx/sites-available
sudo cp defaults api.test.demoserver.com # this can be the url of your api
```
2. **Open the new file in using Vim**
```bash
sudo vim api.test.demoserver.com
```
3. **Make the following changes**
```config
...
server{
...
server_name api.test.demoserver.com # API URL

location / {
		# The local address of your server 
		# make note of the port number
		proxy_pass http://localhost:3000;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
	}
}
```
4. **Add a Symlink to the sites-enabled folder**
```bash
cd ../sites-enabled # I prefer to navigate to the place where i will be creating the link
ln -s ../sites-available/api.test.demoserver.com ./api.test.demoserver.com
```
5. **Restart Nginx**
```bash
sudo nginx -t # This should not return any error
sudo systemctl reload nginx
```
## 9. Setting Up GitHub Actions for CD
### 9.1. For API Server
1. It is recommended to follow `main` <- `dev` <- `feature` flow for CD to prevent unwanted deployment
2. **Open your remote repository on GitHub,** and navigate to _Settings_
3. **Create a self-hosted runner**, this can be found under _Code and automation_ => _Actions_ => _Runners_
4. **Setup your Runner**, select appropriate Runner Image _(The OS you will host the runner on)_and the Architecture
5. **On your machine create a folder named `production-api`**
```bash
mkdir prodution-api # this can be anything you want
cd production-api
```
1. **Copy and paste all of the given code under Download**
2. **Copy the first line of code,** under _Configure_
3. **Instead of the blocking `run.sh` we are going to use the runner service**
```bash
ls -al # there must be a svc.sh file with execute permission
sudo ./svc.sh install # this installs the runner as a service
sudo ./svc.sh start # this starts the runner
```
8. **Setup the YAML for CD automation**, this needs to be on your GitHub repo so start by creating a new branch on GitHub, i named it `setup/cd`
9. **Go to Actions** from the top tab and select the __set up a worflow yourself ->__ option
10. **Make sure that you are on `setup/cd` branch to avoid action trigger**
11. __Copy the following code__
```yaml
# This workflow will do a clean installation of node dependencies, cache/restore them, build the source code and run tests across different versions of node
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-nodejs

name: Demo App API build

on:
  push:
    branches: ["main"]

jobs:
  build:
    runs-on: self-hosted # this needs to be on this file

    strategy:
      matrix:
        node-version: [16.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"
      - run: npm i
      - run: touch .env
      - run: echo '${{secrets.ENV}}' >> .env # look below for info on this
      - name: Start API Service
        run: |
          pm2 start server/app.js
```
12. **Commit the changes**
13. **Trigger deployment**, this can be done by opening a PR from `setup/cd` => `dev` => `main`
14. If you are running the action for the first time you may want to check the __Actions__ tab and verify that there are no errors.

### 9.2. For Frontend App
1. **Follow all of the steps mentioned above** but replace the yaml file with this
```yaml
# This workflow will do a clean installation of node dependencies, cache/restore them, build the source code and run tests across different versions of node
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-nodejs

name: Demo Frontend Build

on:
  push:
    branches: ["main"]

jobs:
  build:
    runs-on: self-hosted

    strategy:
      matrix:
        node-version: [16.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"
      - run: npm i
      - run: touch .env
      - run: echo '${{secrets.ENV}}' >> .env
      - run: npm run build
      - name: Copy dist to /var/www/html
        run: |
          sudo cp -R dist/* /var/www/html
```
## 10. Using Environment Variables in Actions
1. **Add environment variables as secret**, go to _Settings_ => _Security_ => _Secrets and variables_ => _Actions_
2. **Create a new secret**, this can be done by clicking the _New repository secret_ button.
3. **Paste your .env contents**, set the name to `ENV` 
## 11 Aliasing Multipass with an actual domain name
1. **Open the `hosts` file**
```bash
sudo vim /etc/hosts
```
2. **Add the alias and IP address**, this need to go below every other alias you have setup, The IP needs to be the one from the Multipass List
```hosts
10.169.38.66      test.demoserver.com
```
3. **Test if the alias worked**
```bash
curl 'http://test.demoserver.com'
```
And the response should be,
![[Pasted image 20230828103307.png]]
For any other response, consult docs or support

