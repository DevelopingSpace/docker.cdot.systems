## What is this? ##
CDOT and Developing space organisations use docker containers to run our applications. Once our project is ready to be published, we can "pack it" in a single docker container. Every developer can then take this same container, and run it with all of its components working consistently on their machine. We also use containers to "pack" our applications for a hosted live version. So, when we have developed a new version of the application - all we have to do is replace the old container build with the new one. 

This repository is a public collection of such containers, so that everyone is able to run our application builds.

# docker.cdot.systems

This is the configuration for the CDOT Docker Registry, available at https://docker.cdot.systems.

## Usage

Anyone can `pull` Docker images from the registry.  For example, to `pull` an image named `example` you would do:

```sh
$ docker pull docker.cdot.systems/example
```

In order to `push` Docker images, you must first authenticate:

```sh
$ docker login https://docker.cdot.systems -u <username>
Password:........
Login Succeeded
```

You can now `tag` your images with `docker.cdot.systems/<repo>:<tag>` and `push`:

```sh
$ docker build -t docker.cdot.systems/example:v1.0.5
$ docker push docker.cdot.systems/example:v1.0.5
```

## Administration

### Setup

The server is run out of `/usr/local/src/docker.cdot.systems`.  It requires you to install:

- git
- docker
- httpd-tools

The `docker_auth` configuration needs to be defined in `config/docker_auth/auth_config.yml`.  An example config file is available at [config/docker_auth/auth_config_example.yml](config/docker_auth/auth_config_example.yml).  Start by copying it to `config/docker_auth/auth_config.yml`:

```sh
$ cp config/docker_auth/auth_config_example.yml config/docker_auth/auth_config.yml
```

### Running the Server

To start the server, use:

```sh
$ cd /usr/local/src/docker.cdot.systems
$ docker-compose up -d
```

To see logs for any of the services defined in [docker-compose.yaml](docker-compose.yaml), use:

```sh
$ docker ps
# find your desired container...
$ docker logs -f <container>
```

To stop the server, use:

```sh
$ cd /usr/local/src/docker.cdot.systems
$ docker-compose down
```

### Accounts

Update the `users` and `acl` sections of `config/docker_auth/auth_config.yml` in order to create your users:

```yml
users:
  # Passwords are specified as a BCrypt hash. Use htpasswd to generate them.
  'admin':
    password: '$2y$05$LO.vzwpWC5LZGqThvEfznu8qhb5SGqvBSWY1J3yZ4AxtMRZ3kN5jC'  # badmin
  'test':
    password: '$2y$05$WuwBasGDAgr.QCbGIjKJaep4dhxeai9gNZdmBnQXqpKly57oNutya' # 123
  '': {} # Allow anonymous (no "docker login") access for pulling images (see acl below).

acl:
  - match: { account: 'admin' }
    actions: ['*']
    comment: 'Admin has full access to everything.'
  - match: { account: 'test' }
    actions: ['push', 'pull']
    comment: 'Test account has push and pull access'
  - match: { account: '' }
    actions: ['pull']
    comment: 'Any anonymous user has pull access'
```

### Adding/Modifying User Accounts

To create a new user/password pair, generate a hash for the user's password. For example:

```sh
$ htpasswd -n -B -b -C 10 test-user 1234
test-user:$2y$10$Sx4ERcQPJ9z8PY5MjWTus.0tdL17o/VokiM7oPe8aRshsvL1dwRJC
```

Update `config/docker_auth/auth_config.yml` to include the user under `users`:

```yml
users:
  'test-user':
    password: '$2y$10$Sx4ERcQPJ9z8PY5MjWTus.0tdL17o/VokiM7oPe8aRshsvL1dwRJC'
```

Update the permissions for this user under `acl` (see [ACLs doc reference](https://github.com/cesanta/docker_auth/blob/4d80afcb425e5df9e84b6e9c9ba51689394cc8b8/examples/reference.yml#L311-L396)):

```yml
acl:
  - match: { account: 'test-user' }
    actions: ['push', 'pull']
    comment: 'test-user has push and pull access'
```

Restart the server:

```sh
$ cd /usr/local/src/docker.cdot.systems
$ docker-compose restart
```
