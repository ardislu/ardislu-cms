# [ardislu-cms](https://cms.ardis.lu)

This repo contains the code for the Content Management System (CMS) for my personal website [ardis.lu](https://ardis.lu). Powered by [Strapi](https://strapi.io/).

## Quick Overview

Strapi will spin up a SQLite database, RESTful API surface to read/write to the database, and an admin front-end for non-developers to manage content. Strapi is written with node.js.

## Initial Setup

I'm hosting this on a free [Google Compute Engine](https://cloud.google.com/compute) f1-micro instance. Before getting to the Strapi server, we need to configure the virtual machine. 

1. Create a Compute Engine f1-micro instance. 
- Select `Ubuntu 20.04 LTS` (or the latest LTS) as the OS. 
- Check `Allow HTTP traffic` and `Allow HTTPS traffic`.
- In `Network interfaces`, get a static external IP (i.e., not ephemeral).

2. Go to the [GCP Console](https://console.cloud.google.com/) to SSH into the instance.

3. Install [nvm](https://github.com/nvm-sh/nvm) (see the linked GitHub repo to get the URL for the latest version of `nvm`):
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```

4. Use `nvm` to install and use the latest version of `node` (change `14.16.1` to the latest version):
```
nvm ls-remote # Show all node versions
nvm install 14.16.1 # Install the latest version
nvm use 14.16.1 # Configure PATH to this installation of node
nvm alias default 14.16.1 # Automatically do 'nvm use 14.16.1' on future sessions 
```
(Optional) You can update `npm` independently of `node`:
```
nvm install-latest-npm
```

5. Confirm successful installation:
```
node -v && npm -v
```

6. Install global node packages:
```
npm i -g yarn strapi pm2
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
sudo env PATH=$PATH NODE_ENV=production pm2 start strapi --name "ardislu-cms" -- start
```

Command explanation:
- `sudo`: run the following command as superuser (required to expose the web server to the internet)
- `env`: set the following environment variables
- `PATH=$PATH`: set the superuser PATH to the same as the user's PATH - required to use `pm2` and `strapi` (by default, the `sudo` command uses a separately-configured PATH)
- `NODE_ENV=production`: tell `strapi` to use production settings
- `pm2 start strapi --name "ardislu-cms"`: initialize the `pm2` process with the `strapi` command and the name "ardislu-cms"
- `-- start`: pass `start` to the `strapi` command (the space after `--` is required to pass it "down" to `strapi` instead of to `pm2`)

4. Configure pm2 to restart the process on startup:
```
sudo env PATH=$PATH pm2 startup
sudo env PATH=$PATH pm2 save
```

## Updating the data model
You need to run Strapi in `develop` mode to actually update any content using the admin front-end. 

1. Pause the `pm2` production process:
```
sudo env PATH=$PATH pm2 stop 0
```

2. Run Strapi in development:
```
cd ardislu-cms
sudo env PATH=$PATH strapi develop
```

3. After finishing content updates, restart the production process:
```
sudo env PATH=$PATH pm2 start 0
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
sudo env PATH=$PATH pm2 stop 0
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
sudo env PATH=$PATH pm2 start 0
```

## Kill the pm2 process
You can force kill the `pm2` process with this command (useful when using different versions of `node` with multiple global `pm2` installations):
```
sudo pkill -f PM2
```
