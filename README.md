# Install ERP Next 11 on latest Frappe Docker

## Prerequisites

- Git
- Docker
- User Added to Docker Group


## Bootstrap Containers for development

Clone and change directory to frappe_docker directory

```shell
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```

Copy example devcontainer config from `devcontainer-example` to `.devcontainer`

```shell
cp -R devcontainer-example .devcontainer
```

Copy example vscode config for devcontainer from `development/vscode-example` to `development/.vscode`.

```shell
cp -R development/vscode-example development/.vscode
```

## Use VSCode Remote Containers extension

Use [VSCode Remote - Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).


### Setup first bench

Downgrade Node Version using NVM, because ERPNext only support Node Version 10
```shell
nvm use v10.24.1
```

Run the following commands in the terminal inside the container. You might need to create a new terminal in VSCode.

```shell
bench init --skip-redis-config-generation --frappe-branch version-11 frappe-bench
cd frappe-bench
```

If got this Error, don't rollback the init step:
```shell
subprocess.CalledProcessError: Command 'supervisorctl status' returned non-zero exit status 127.

ERROR: There was a problem while creating frappe-bench
Do you want to rollback these changes? [y/N]: n
```

Retry unfinished setup, to get all dependencies installed completely
```
bench setup requirements
bench update --requirements
```
### Setup hosts

We need to tell bench to use the right containers instead of localhost. Run the following commands inside the container:

```shell
bench set-mariadb-host mariadb
bench set-redis-cache-host redis-cache:6379
bench set-redis-queue-host redis-queue:6379
bench set-redis-socketio-host redis-socketio:6379
```

### Create a new site with bench

We will get error on command: ```bench new-site```, because we have different frappe and bench, so we need install ```frappe-bench``` package temporary on virtual environment to get command ```bench new-site``` work perfectly.

Activate environment
```shell
source env/bin/activate
```
Install latest bench (It will got error, because some dependencies conflict. No problem, frappe-bench will be removed after we create our site)
```shell
pip install frappe-bench
```
Create new Site
```shell
bench new-site mysite.localhost --mariadb-root-password 123 --admin-password admin --no-mariadb-socket
```
To install custom app

```shell
bench get https://github.com/myusername/myapp
bench --site mysite.localhost install-app myapp
```

Remove ```frappe-bench``` package to avoid dependencies conflicts
```shell
pip uninstall frappe-bench
```

### Start Frappe without debugging

Execute following command from the `frappe-bench` directory.

```shell
bench start
```

If face an issue related asset not loaded, we need build the asset:
```shell
bench build
bench migrate
```

