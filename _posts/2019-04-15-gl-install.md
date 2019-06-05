---
layout: page
title: "Install"
category: greenlight
date: 2019-04-16 16:29:25
---

# Installing on a BigBlueButton Server

To make Greenlight as easy to install as possible, we've created a Docker image that wraps the application. It is **highly** recommended that you use Docker when install Greenlight on a BigBlueButton server.

Following this installation process will install Greenlight with the default settings. Through the [Administrator Interface](gl-admin.html), you can customize the Branding Image, Primary Color and other Site Settings. If you would like to check out the Greenlight source code and make changes to it, follow these [installation instructions](gl-customize.html#customizing-greenlight)

You should run all commands in this section as `root` on your BigBlueButton server.

## BigBlueButton Server Requirements

Before you install Greenlight, you must have a BigBlueButton server to install it on. This server must:

* have a version of BigBlueButton 2.0 (or greater).
* have a fully qualified hostname.
* have a valid SSL certificate (HTTPS).

## 1. Install Docker

The official Docker documentation is the best resource for Docker install steps. To install Docker (we recommend installing Docker CE unless you have a subscription to Docker EE), see [Install Docker on Ubuntu](https://docs.docker.com/engine/installation/linux/ubuntu/).

Before moving onto the next step, verify that Docker is installed by running:

```
docker -v
```

## 2. Install Greenlight

First, create the Greenlight directory for its configuration to live in.

```
mkdir ~/greenlight && cd ~/greenlight
```

Greenlight will read its environment configuration from the `.env` file. To generate this file and install the Greenlight Docker image, run:

```
docker run --rm bigbluebutton/greenlight:v2 cat ./sample.env > .env
```

## 3. Configure Greenlight

If you open the `.env` file you'll see that it contains information for all of the Greenlight configuration options. Some of these are mandatory.

When you installed in step two, the `.env` file was generated at `~/greenlight/.env`.

### Generating a Secret Key

Greenlight needs a secret key in order to run in production. To generate this, run:

```
docker run --rm bigbluebutton/greenlight:v2 bundle exec rake secret
```

Inside your `.env` file, set the `SECRET_KEY_BASE` option to this key. You don't need to surround it in quotations.

### Setting BigBlueButton Credentials

By default, your Greenlight instance will automatically connect to `test-install.blindsidenetworks.com` if no BigBlueButton credentials are specified. To set Greenlight to connect to your BigBlueButton server (the one it's installed on), you need to give Greenlight the endpoint and the secret. To get the credentials, run:

```
bbb-conf --secret
```

In your `.env` file, set the `BIGBLUEBUTTON_ENDPOINT` to the URL, and set `BIGBLUEBUTTON_SECRET` to the secret.

### Verifying Configuration

Once you have finished setting the environment variables above in your `.env` file, to verify that you configuration is valid, run:

```
docker run --rm --env-file .env bigbluebutton/greenlight:v2 bundle exec rake conf:check
```

If you have configured an SMTP server in your `.env` file, then all four tests must pass before you proceed. If you have not configured an SMTP server, then only the first three tests must pass before you proceed.

## 4. Configure Nginx to Route To Greenlight

Greenlight will be configured to deploy at the `/b` subdirectory. This is necessary so it doesn't conflict with the other BigBlueButton components. The Nginx configuration for this subdirectory is stored in the Greenlight image. To add this configuration file to your BigBlueButton server, run:

```
docker run --rm bigbluebutton/greenlight:v2 cat ./greenlight.nginx | sudo tee /etc/bigbluebutton/nginx/greenlight.nginx
```

Verify that the Nginx configuration file (`/etc/bigbluebutton/nginx/greenlight.nginx`) is in place. If it is, restart Nginx so it picks up the new configuration.

```
systemctl restart nginx
```

This will routes all requests to `https://<hostname>/b` to the Greenlight application. If you wish to use a different relative root, you can follow the steps outlined [here](gl-customize.html#using-a-different-relative-root).

Optionally, if you wish to have the default landing page at the root of your BigBlueButton server redirect to Greenlight, add the following entry to the bottom of `/etc/nginx/sites-available/bigbluebutton` just before the last `}` character.

```
location = / {
  return 307 /b;
}
```

To have this change take effect, you must once again restart Nginx.

## 5. Start Greenlight 2.0

To start the Greenlight Docker containter, you must install `docker-compose`, which simplifies the start and stop process for Docker containers.

Install `docker-compose` by following the steps for installing on Linux in the [Docker documentation](https://docs.docker.com/compose/install/). You may be required to run all `docker-compose` commands using sudo. If you wish to change this, check out [managing docker as a non-root user](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user).


### Using `docker-compose`

Before you continue, verify that you have `docker-compose` installed by running:

```
docker-compose -v
```

Next, you should copy the `docker-compose.yml` file from the Greenlight image in to `~/greenlight` directory. To do this, run:

```
docker run --rm bigbluebutton/greenlight:v2 cat ./docker-compose.yml > docker-compose.yml
```

Once you have this file, from the `~/greenlight` directory, start the application using:

```
docker-compose up -d
```

This will start Greenlight, and you should be able to access it at `https://<hostname>/b`.

The database is saved to the BigBlueButton server so data persists when you restart. This can be found at `~/greenlight/db`.

All of the logs from the application are also saved to the BigBlueButton server, which can be found at `~/greenlight/log`.

If you don't wish for either of these to persist, simply remove the volumes from the `docker-compose.yml` file.

To stop the application, run:

```
docker-compose down
```

### Using `docker run`

`docker run` is no longer the recommended way to start Greenlight. Please use `docker-compose`.

If you are currently using `docker run` and want to switch to `docker-compose`, follow these [instructions](#switching-from-docker-run-to-docker-compose).

# Applying `.env` Changes

After you edit the `.env` file, you are required to restart Greenlight in order for it to pick up the changes. Ensure you are in the Greenlight directory when restarting Greenlight.

## If you ran Greenlight using `docker-compose`

Bring down Greenlight using:
```
docker-compose down
```
then, bring it back up.
```
docker-compose up -d
```

## If you ran Greenlight using `docker run`

`docker run` is no longer the recommended way to start Greenlight. Please use `docker-compose`.

If you are currently using `docker run` and want to switch to `docker-compose`, follow these [instructions](#switching-from-docker-run-to-docker-compose).

# Updating Greenlight

To update Greenlight, all you need to do is pull the latest image from [Dockerhub](https://hub.docker.com/).

```
docker pull bigbluebutton/greenlight:v2
```

Then, [restart Greenlight](#applying-env-changes).

# Switching from `docker run` to `docker-compose`
To switch from using `docker run` to start Greenlight, to using `docker-compose`, run the following commands:
```
docker stop greenlight-v2
docker rm greenlight-v2
```

And then follow the instructions for [Starting Greenlight](#5-start-greenlight-20) 


# Troubleshooting Greenlight

Sometimes there are missteps and incompatibility issues when setting up applications.

## Changes not appearing

If you made changes to the `.env` file, you will need to [restart Greenlight](#applying-env-changes) to see the changes appear.

## Checking the Logs

The best way for determining the root cause of issues in your Greenlight application is to check the logs.

Docker is always running on a production environment, so the logs will be located in `log/production.log` from the `~/greenlight` directory.