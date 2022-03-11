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

The server is run out of `/usr/local/src/docker.cdot.systems`.

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

## Accounts

Docker accounts are defined in [config/docker_auth/auth_config.yml](config/docker_auth/auth_config.yml) like so:

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

Generate a new hash for the user's password (e.g., `user=test-user`, `password=1234`):

```sh
$ htpasswd -n -B -b -C 10 test-user 1234
test-user:$2y$10$Sx4ERcQPJ9z8PY5MjWTus.0tdL17o/VokiM7oPe8aRshsvL1dwRJC
```

Update [config/docker_auth/auth_config.yml](config/docker_auth/auth_config.yml) to include the user under `users`:

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
