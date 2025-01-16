**This is a fork of [serversideup/github-action-docker-swarm-deploy](https://github.com/serversideup/github-action-docker-swarm-deploy) to allow multiple managers in a Docker Swarm cluster.**

<p align="center">
		<img src=".github/img/readme-header.png" width="1280" alt="Header Image">
</p>
<p align="center">
	<a href="https://github.com/serversideup/github-action-docker-swarm-deploy/blob/main/LICENSE" target="_blank"><img src="https://badgen.net/github/license/serversideup/github-action-docker-swarm-deploy" alt="License"></a>
	<a href="https://github.com/sponsors/serversideup"><img src="https://badgen.net/badge/icon/Support%20Us?label=GitHub%20Sponsors&color=orange" alt="Support us"></a>
  <br />
  <a href="https://community.serversideup.net"><img alt="Discourse users" src="https://img.shields.io/discourse/users?color=blue&server=https%3A%2F%2Fcommunity.serversideup.net"></a>
  <a href="https://serversideup.net/discord"><img alt="Discord" src="https://img.shields.io/discord/910287105714954251?color=blueviolet"></a>
</p>

## Introduction
This is a GitHub Action intended to simplify the deployment experience with GitHub Actions + Docker Swarm.

### Features:
- 😃 Simple to use
- 🏗️ Bring your own container + configuration
- 💯 Replicate 100% of production from Development to CI to Deployment
- 💪 Use with self-hosted registries
- 🧮 Store MD5 hashes in environment variables for deployment
- 🔐 Use with private registries
- 🏠 Use .env files for deployment
- 🌐 Works in a cluster with multiple managers

## Usage
Here is an example workflow:

```yml
name: Production Deployment
on:
  push:
    branches:
      - main

jobs:
  deploy:
    needs: build
    runs-on: ubuntu-24.04
    steps:
      - uses: harrisonratcliffe/github-action-docker-swarm-deploy@v1.0.0
        with:
          ssh_deploy_private_key: "${{ secrets.SSH_DEPLOY_PRIVATE_KEY }}"
          ssh_remote_hostnames: "${{ secrets.SSH_REMOTE_HOSTNAMES }}"
          registry: "ghcr.io"
          registry-username: "${{ github.actor }}"
          registry-token: "${{ secrets.GITHUB_TOKEN }}"
          stack_name: "${{ env.PROJECT_NAME }}"
          md5_file_path: "./.infrastructure/conf/traefik/prod/traefik.yml"
          md5_variable_name: "SPIN_MD5_HASH_TRAEFIK_YML"
          env_file_base64: "${{ secrets.ENV_FILE_BASE64 }}"
```
## How to use the action

The following inputs are available:

| Parameter               | Description                                                                                     | Default                                              | Required |
|-------------------------|--------------------------------------------------------------------------------------------------|------------------------------------------------------|----------|
| docker_compose_file_path| Set your docker compose file path with the CLI options.                                          | `-c docker-compose.yml -c docker-compose.prod.yml`   | false    |
| env_file_base64         | The base64 encoded .env file to load into the container.                                         |                                                      | false    |
| log_level               | The log level to use for the Docker CLI.                                                         | `debug`                                              | false    |
| md5_file_path           | Set the path to the file you would like to get the MD5 checksum for.                             |                                                      | false    |
| md5_variable_name       | Set the name of the variable to store the MD5 checksum in.                                       | `MD5_CHECKSUM`                                       | false    |
| registry                | Comma-separated list of container registries to authenticate with (e.g., "docker.io,ghcr.io").   | `docker.io`                                          | false    |
| registry-token          | The token or password to use to authenticate with the container registry.                        |                                                      | ⚠️ true  |
| registry-username       | The username to use to authenticate with the container registry.                                 |                                                      | ⚠️ true  |
| ssh_deploy_private_key  | The private key you have authenticated to connect to your servers via SSH.                        |                                                      | ⚠️ true  |
| ssh_deploy_user         | The user that you would like to connect as on the remote servers via SSH.                         | `deploy`                                             | ⚠️ true  |
| ssh_remote_hostnames     | The hostnames or IP addresses (commas separated) of the servers you want to connect to.         |                                                      | ⚠️ true  |
| ssh_remote_known_hosts  | The public key of your SSH servers to validate we are connecting to the right server.             |                                                      | false    |
| ssh_remote_port         | The SSH port of the remote servers you would like to connect to.                                  | `22`                                                 | false    |
| stack_name              | The name of your Docker stack.                                                                   |                                                      | ⚠️ true  |

## Working with SSH
SSH can have a few moving parts and it's important you get this right. Here's a few pointers to ensure you have the right setup.

### ssh_deploy_user
This is the user you want to connect to your servers with. This is most likely going to be `deploy` but could be different depending on your setup. It is very important that whatever user you choose, this user should have permissions to run `docker stack deploy` (without `sudo`).

### ssh_remote_hostnames
This is the hostnames or IP addresses of your servers. This is most likely going to be your server's public IP address. This can be `1.2.3.4` or `myserver.example.com`. For multiple manager nodes you can commas separate them like so: `1.2.3.4,5.6.7.8,myserver.example.com`

### ssh_remote_port
This is the port of your SSH servers. This is most likely going to be `22` but could be different depending on your setup. Make sure this port is accessible from GitHub Actions. You may have to allow this port through your router, firewall, or security policy with your hosting provider. Currently, the port must be the same on all servers if you are using a Docker Swarm cluster.

### ssh_deploy_private_key
This is the private key you use to authenticate to your servers via SSH. It must be in a valid private key format. 

To generate a keypair, you can use the following commands:

```bash
ssh-keygen -o -a 100 -t ed25519 -f ~/Desktop/id_ed25519_deploy -C deploy
```
This will create two files on your desktop. You can use `cat` to get the content of your files.

> [!WARNING]  
> Be sure you're not copying hidden characters or extra whitespaces

```bash
cat ~/Desktop/id_ed25519_deploy # Get the content of your PRIVATE key
cat ~/Desktop/id_ed25519_deploy.pub # Get the content of your PUBLIC key

## If you use macOS, you can use `pbcopy` to copy your key to your clipboard

cat ~/Desktop/id_ed25519_deploy | pbcopy # Copy your private key to your clipboard
cat ~/Desktop/id_ed25519_deploy.pub | pbcopy # Copy your public key to your clipboard
```

> [!CAUTION]
> In order for you to connect to your servers, the user you're connecting as must have your public key in their **authorized_keys** file.

Copy the output and add it to the `~/.ssh/authorized_keys` file on your servers for the user you're connecting as.

### ssh_remote_known_hosts
This is the public key of your SSH servers to validate we are connecting to the right server. It must be in a [valid known_hosts format](https://www.ibm.com/docs/en/zos/3.1.0?topic=daemon-ssh-known-hosts-file-format).

### Removing the "ssh_remote_known_hosts" warning
![image](.github/img/known-hosts-warning.png)
For simplicity sake, we will automatically scan the known public SSH keys of your servers and attempt to make a connection. The problem with this is it opens you up to a man-in-the-middle attack.

To ensure you're validating the identity of your servers, you can set the `ssh_remote_known_hosts` input with the public key of your servers. You can set this value to a GitHub secret like `SSH_REMOTE_KNOWN_HOSTS`:

```yml
- uses: harrisonratcliffe/github-action-docker-swarm-deploy@v1.0.0
  with:
    registry-token: "${{ secrets.GITHUB_TOKEN }}"
    registry-username: "${{ github.actor }}"
    ssh_deploy_private_key: "${{ secrets.SSH_DEPLOY_PRIVATE_KEY }}"
    ssh_remote_hostnames: "${{ secrets.SSH_REMOTE_HOSTNAME }}"
    ssh_remote_known_hosts: "${{ secrets.SSH_REMOTE_KNOWN_HOSTS }}" # Set "SSH_REMOTE_KNOWN_HOSTS" in GitHub Actions Secrets 
    stack_name: "${{ env.PROJECT_NAME }}"
  env:
    SPIN_IMAGE_DOCKERFILE_PHP: "ghcr.io/${{ github.repository }}:${{ github.sha }}"
    SPIN_DEPLOYMENT_ENVIRONMENT: production
```
#### Setting the "ssh_remote_known_hosts" secret
![image](.github/img/secrets.png)

You can set the `ssh_remote_known_hosts` secret by getting the public key of your servers and setting it as a GitHub secret. You can run this from your **local machine** (not in GitHub Actions) to get the public key of your servers:

> [!NOTE]  
> Replace `myserver.example.com` with the hostname of your server. You can also change the port by changing `-p 22` to your desired port.

```bash
ssh-keyscan -p 22 -H myserver.example.com 2>/dev/null | sort -u
```

The output will look similar to this, with hashed hostnames:

```
|1|BfvcToAPMeAK0zR9FShCnP5CCaw=|ltUazkjjoIKsoBFQMF5yOTJt/Ks= ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBKIAz4U9GvgyBttgCnvi4AfBq3CdQ9XqAryrIyO1O60
|1|J/BpMKspk0BwPAxR28Dzc7gVGgw=|RAimV4/7iS4jlmFmDAfex/nKDUQ= ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQChvNZNpcjdSXJWVdnhieQXRgBVUUwpexLz0dbDegUj68vrzXsgtrnGtf+sJlRhI6C7jBZDfxk2jXL1ASfxEQUqbvptZTG68uusD1DYx3wtb/kTqvJ3JkFuWJbt2zLyZktPrueHA9cvuquW46M6wSZN5AZddNitUZ09Bpb+dTVZkjbEDOiGoHRDj5M86e1rr/8UGNrAVZl/hckup3lfu3B3P0LKnGnMw+/DXIKvJiwVJ3OdHzyq6D/x9uNgcOUA7UPgUbV30gyFtWr2Az6Vn/ZolDOGasK9iI5WjvBdXwyWNwEnnR539RutiwbS/XTnb0Jj/fFS5NM2/AM3nCT37D4uQA7aJFka7keUTJZJIVanziz9Ty76lloweLDKHN2CyvUijjSx5HaqV9Dr2nTefTPPvzz1D9xU0WJX8KC77Wcu8qEjqSwNihJqucXQvq4xeBZ85OGPbvzAFYqZdjynzVsLP50E7kmdaW3VJx88hbg+vyXrJD1urcOVPNtGoMpN2Mc=
|1|cYUT42KbDx+rQD0YSpKgDxbFWBc=|D13n4gWdMyJ+C2nifoEEeFmezmE= ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBN7wgADkhTHi7WER2pCZ5/10HBmhSIAq9zS1rWiBG5A8t2ATh5QnJ17XtPKXEJGPH8nogry/bZ+WKxI4zojGD+Q=
```

Copy the output and set it as a GitHub secret (usually called `SSH_REMOTE_KNOWN_HOSTS`).

![image](.github/img/secrets.png)

#### Validate the known hosts file
If you need to validate the known hosts file, you can save it in a file on your local machine and attempt to SSH into your servers with it:

```bash
ssh -p 22 -i /path/to/test_known_hosts_file myserver.example.com
```

If you cannot connect from your local machine, then you know there is an issue with the known hosts file itself.

## Advanced Usage
We also have some helpful features for our power users out there.

### Getting the MD5 Checksum of a file
We include an optional input to get the MD5 checksum of a file. This is useful if you're working with Docker Configs and you only want the service to update if the file has changed. You just need to set the following inputs (a full example is available at the top of this document):

```yml
steps:
  - uses: harrisonratcliffe/github-action-docker-swarm-deploy@v1.0.0
    with:
      md5_file_path: "./path/to/my/file.txt"
      md5_variable_name: "MY_FILE_MD5"
```

This will store the MD5 checksum of the file at `./path/to/my/file.txt` in the environment variable `MY_FILE_MD5`.

### Using an .env file
You can use an .env file to set environment variables for your container. This is useful if you're working with environment variables that are different for each environment. You can set the .env file as a base64 encoded string and it will be decoded and loaded into the container.

```yml
steps:
  - uses: harrisonratcliffe/github-action-docker-swarm-deploy@v1.0.0
    with:
      env_file_base64: "${{ secrets.ENV_FILE_BASE64 }}"
```

To set the value of `ENV_FILE_BASE64`, you can use the following command:

```bash
cat .env | base64
```

Any variable set in the .env file will be available to the deployment to be used in your docker-compose.yml file.

For example, if you have a .env file with the following:

```
DB_HOST=mysql
DB_PORT=3306
DB_USER=root
DB_PASSWORD=password
```

Then you can use the following in your docker-compose.yml file: 

```yml
services:
  mysql:
    environment:
      - DB_HOST=${{ env.DB_HOST }}
      - DB_PORT=${{ env.DB_PORT }}
      - DB_USER=${{ env.DB_USER }}
      - DB_PASSWORD=${{ env.DB_PASSWORD }}
```
