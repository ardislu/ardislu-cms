# [ardislu-cms](https://cms.ardis.lu)

This repo contains the code for the Content Management System (CMS) for my personal website [ardis.lu](https://ardis.lu). Powered by [Strapi](https://strapi.io/).

## Quick Overview

Included in the repo is a SQLite database, RESTful API surface to read/write to the database, and an admin front-end for non-developers to manage content. All written in node.js.

## Deployment

I'm hosting this on a free [Google Compute Engine](https://console.cloud.google.com/) f1-micro instance. The instance is running Ubuntu 18.04 LTS with PM2 Runtime to host Strapi. Go to the GCP Console to SSH into the instance.

