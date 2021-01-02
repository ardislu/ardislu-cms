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

3. Install [nvm](https://github.com/nvm-sh/nvm) (see GitHub repo to get the URL for the latest version of `nvm`):
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash
```

4. Use `nvm` to install the latest version of `node`:
```
nvm install node
```

5. Confirm successful installation:
```
node -v && npm -v
```

6. Create and change npm's default directory:
```
cd ~
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
```

7. Create (or modify) a ~/.profile file and add this line:
```
sudo vim ~/.profile
```
Add these lines to the bottom of ~/.profile:
```
# set PATH so global node modules install without permission issues
export PATH=~/.npm-global/bin:$PATH
```

8. Update system variables:
```
source ~/.profile
```

9. Install global node packages:
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

## Updating the data model
You need to run Strapi in `develop` mode to actually update any content using the admin front-end. 

1. Pause the `pm2` production process:
```
pm2 stop 0
```

2. Run Strapi in development:
```
cd ardislu-cms
sudo npm run develop
```

3. After finishing content updates, restart the production process:
```
pm2 start 0
```

## Rebuilding the CMS

The f1-micro instance does not have enough provisioned RAM to build Strapi locally. Instead, build it remotely (e.g. on your own development computer), then push the build files to GitHub. Then pull the compiled files on the virtual machine and restart the server.

### On the development computer:

1. Use `npm-check-updates` to check (and update) `package.json` dependencies to latest versions:
```
npx ncu
```

2. Install updated dependencies:
```
yarn install
```

3. Rebuild the CMS:
```
yarn build
```

4. Commit and push changes (including the `build` folder!)

### On the virtual machine:

1. Pause the `pm2` process:
```
pm2 stop 0
```

2. Pull latest changes and the latest build:
```
git pull
```

3. Update dependencies per updated `package.json`:
```
yarn install
```

4. Restart the pm2 process with the updated files:
```
pm2 start 0
```