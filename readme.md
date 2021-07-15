# Guide to set up Dokku with Digitalocean from scratch

### Set up digitalocean droplet

1. Select plan eg Ubuntu, basic
1. Click on 'Add public SSH key'
   1. Create new SSH key with `ssh-keygen` and save to `/Users/USER/.ssh`
   1. Paste contents of public key into field and name your key

### Set up environment on linux

[Follow docs](https://dokku.com/docs/getting-started/installation/#1-install-dokku)

1. Install Dokku

```
wget https://raw.githubusercontent.com/dokku/dokku/v0.24.10/bootstrap.sh;
sudo DOKKU_TAG=v0.24.10 bash bootstrap.sh
```

2. Go to server's IP address and finish set up for Dokku

```
open http://<SERVER_IP>
```

### Set up key on local machine

1. Open Git Bash and go to ssh folder

```
cd ~/.ssh
```

2. In `config` file (create one if not found), write:

```
#account- dokku
Host <SERVER_IP_ADDRESS>
 HostName <SERVER_IP_ADDRESS>
 IdentityFile ~/.ssh/<PRIVATE_KEY_FILE_NAME>
```

### Create Node.js app

1. Create express app that listens on port 4000

2. Create Dockerfile

```
FROM node:15
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
ENV PORT 4000
CMD ["node", "./index.js"]
```

### Set up domain on Namecheap (or another provider)

1. Purchase domain on namecheap
2. On digitalocean, go to `Networking` > your app > `Manage`
3. On namecheap, go to `Dashboard` > your domain > `Manage` > `Nameservers` > select `Custom DNS`
4. Copy nameservers from digitalocean panel to namecheap's custom DNS

```
ns1.digitalocean.com.
ns2.digitalocean.com.
ns3.digitalocean.com.
```

5. Wait a few hours for it to be set

### Create dokku application on server

SSH into server

```
ssh root@<SERVER_IP>
```

Create dokku app

```
dokku apps:create <APP_NAME>
```

(Optional) Set env vars

```
dokku config:set <APP_NAME> KEY1=VALUE1 KEY2=VALUE2
```

### Configure dokku application on server

Configure domain

```
dokku domains:add <APP_NAME> <DOMAIN_NAME>
```

Check if domain is set

```
dokku domains:report
```

Remove old hostname

```
dokku domains:remove <APP_NAME> <APP_NAME>.dokku-s-1vcpu-1gb-fra1-01
```

TLS/SSL certificates

```
dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
```

Map container port from 80 to 4000

```
dokku proxy:ports-add <APP_NAME> http:80:<CONTAINER_PORT>
```

Config Letsencrypt email

```
dokku config:set --no-restart <APP_NAME> DOKKU_LETSENCRYPT_EMAIL=<EMAIL>
```

Enable Letsencrypt

```
dokku letsencrypt:enable <APP_NAME>
```

### Deploy application

On local machine

```
git remote add dokku dokku@<SERVER_IP>:<APP_NAME>
```

Push to master

```
git push dokku master
```
