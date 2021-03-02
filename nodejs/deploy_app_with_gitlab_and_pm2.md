Deploy nodejs app with gitlab.com and pm2
=========================================

This manual is about setting up an automatic deploy workflow using [nodejs](https://nodejs.org/en/),
[PM2](http://pm2.keymetrics.io/), [nginx](https://nginx.org/) and
[GitLab CI](https://about.gitlab.com/features/gitlab-ci-cd/).

Configure target server
-----------------------

### 1. Install nodejs and npm

You can find official instruction
[here](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions).

For Ubuntu 16.04 run:

```bash
$curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
$sudo apt install nodejs
```

To check an installation run:

```bash
$ node -v
$ npm -v
```

### 2. Install process manager pm2

[PM2](http://pm2.keymetrics.io/) is a beautiful production process manager for nodejs. It will observe, log and
automatically restart your application if it fall. Run now:

```bash
$ sudo npm install -g pm2@latest
```

To enable auto start pm2 on reboot run:

```bash
$ pm2 startup
```

then **(!important)** follow the instructions on your screen (run displayed command).

### 3. Install git

```bash
$ sudo apt install git
```

If you haven't deploy git keys yet, you should run:

```bash
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

This command will generate private (`/home/dev/.ssh/id_rsa`) and public
(`/home/dev/.ssh/id_rsa.pub`) key. Print public key:

```bash
$ cat /home/dev/.ssh/id_rsa.pub
```

copy it clipboard and paste to gitlab (Edit Profile/SSH Keys, then add an SSH key).

Check ssh access to repository:
```bash
$ ssh -T git@gitlab.com
```

On the question "The authenticity of host...?" answer "yes".
If all is okay, you should see string like *"Welcome to GitLab, yourUsername!"*.

### 4. Generate SSH keys for current server

Now we should generate SSH keys to access current server without password.
Run next command again, but set file path to `/home/dev/access`:

```bash
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

This command will generate private (`/home/dev/access`) and public (`/home/dev/access.pub`) key.
Move new generated key to *authorized_keys*:

```bash
$ cat /home/dev/access.pub >> ~/.ssh/authorized_keys
```

You must copy and save private key on your computer. To print this key on screen use:

```bash
$ cat ~/access
```

### 5. Install nginx

```bash
$ sudo apt install nginx
$ sudo rm /etc/nginx/sites-enabled/default
```

Open nginx config:

```bash
$ sudo nano /etc/nginx/sites-available/myapp
```

and replace it with:

```nginx
server {
  listen 80;
  server_name example.com www.example.com;
  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    roxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
	  proxy_set_header  X-NginX-Proxy 	true;
	  proxy_redirect off;
    proxy_buffering off;
  }
}
```

Then run:

```bash
$ sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/myapp
$ sudo systemctl restart nginx
```

And check nginx status (it should be "active"):

```bash
$ sudo systemctl status nginx
```


Configure deployment with Gitlab CI
-----------------------------------

### 1. Create file `ecosystem.config.js`in root directory of your project:
```bash
$ pm2 ecosystem
```


```javascript
module.exports = {
  /**
   * Application configuration section
   * http://pm2.keymetrics.io/docs/usage/application-declaration/
   */
  apps: [
    {
      name: 'testApp',
      script: 'npm', // script path to pm2 start
      cwd: '/var/www/myapp/current',
      args: 'start',
      // instances: 2, // number process of application
      exec_mode: 'fork_mode',
      autorestart: true, //auto restart if app crashes
      watch: false,
      max_memory_restart: '1G', // restart if it exceeds the amount of memory specified
      // env: {
      //     NODE_ENV: 'development'
      // },
      // env_production: {
      //     NODE_ENV: 'production'
      // }
    }
  ],

  /**
   * Deployment section
   * http://pm2.keymetrics.io/docs/usage/deployment/
   */
  deploy : {
    production: {
      user: 'username_login',
      host: 'ip_server',
      ref: 'origin/master',
      repo: 'git@gitlab.com:username/gitname.git',
      ssh_options: 'StrictHostKeyChecking=no',
      path: '/var/www/myapp',
      'post-deploy': 'npm install --production && pm2 startOrRestart ecosystem.config.js --env=production',
    }
  },
};
```

### 2. Add secret variables in gitlab:

Go to [gitlab.com](https:/gitlab.com) -> Your project -> "Settings" -> "CI/CD" -> "Secret variables".
Add some variables:

| Variable                        | Description                                              |
|---------------------------------|----------------------------------------------------------|
| TARGET_SERVER_SECRET_KEY | Base64 encoded private RSA key to login target server. Make it protected |

### 3. Create file `.gitlab-ci.yml` in root directory of project:

```yaml
image: node:latest

stages:
  - deploy

deploy-production:
  stage: deploy
  before_script:
    - echo "====== Deploy to production server ======"
    - eval "$(ssh-agent -s)"
    - echo "$TARGET_SERVER_SECRET_KEY" | tr -d '\r' | ssh-add - > /dev/null
    - ssh-add -l
    # Install pm2:
    - npm i -g pm2
  script:
    # Delploy
    - echo "Setup target server directories"
    - pm2 deploy ecosystem.config.js production setup 2>&1 || true
    - echo "make deploy"
    - pm2 deploy ecosystem.config.js production
  environment:
    name: deploying
  only:
    - master
```

If all is okay, your project will be automatically deployed every push and merge to master branch.
