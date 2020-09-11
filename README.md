# [ardislu-cms](https://cms.ardis.lu)

This repo contains the code for the Content Management System (CMS) for my personal website [ardis.lu](https://ardis.lu). Powered by [Strapi](https://strapi.io/).

## Quick Overview

Strapi will spin up a SQLite database, RESTful API surface to read/write to the database, and an admin front-end for non-developers to manage content. All written in node.js, in this directory.

## Initial Setup

I'm hosting this on a free [Google Compute Engine](https://cloud.google.com/compute) f1-micro instance. Before getting to the Strapi server, we need to configure the virtual machine. 

1. Create a Compute Engine f1-micro instance. 
- Select `Ubuntu 18.04 LTS` as the OS.
- Check `Allow HTTP traffic` and `Allow HTTPS traffic`.
- In `Network interfaces`, get a static external IP (i.e., not ephemeral).

2. Go to the [GCP Console](https://console.cloud.google.com/) to SSH into the instance.

3. Install Node.js with npm:
```
cd ~
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
...
sudo apt-get install nodejs
...
node -v && npm -v
```

4. Create and change npm's default directory:
```
cd ~
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
```

5. Create (or modify) a ~/.profile file and add this line:
```
sudo vim ~/.profile
```
Add these lines to the bottom of ~/.profile:
```
# set PATH so global node modules install without permission issues
export PATH=~/.npm-global/bin:$PATH
```

6. Update system variables:
```
source ~/.profile
```

7. Install global node packages:
```
npm i -g yarn
npm i -g strapi
npm i -g pm2
```

## Deployment

1. Clone the project files:
```
git clone https://github.com/ardislu/ardislu-cms.git
```

2. `cd` into the cloned directory:
```
cd ardislu-cms
```

2. Install dependencies:
```
yarn install
```

3. Use [pm2](https://pm2.keymetrics.io/) to initially daemonize the Strapi server and run it in the background:
```
sudo env PATH=$PATH NODE_ENV=production pm2 start npm --name "ardislu-cms" -- run start
```

4. Configure pm2 to restart the process on startup:
```
cd ~
pm2 startup
pm2 save
```

## Rebuilding the CMS

The f1-micro instance does not have enough provisioned RAM to build Strapi locally. Instead, build it remotely (e.g. on your own development computer), then push the build files to GitHub. Then pull the compiled files on the virtual machine and restart the server.