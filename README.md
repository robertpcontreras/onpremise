# Sentry Setup on RHEL 7

This post will guide you through the installation process to have a fully working instance of sentry on an instance of Red Hat Enterprise Linux 7.

The recommended approach to installing sentry is by using docker containers to quickly and easily set up all the processes required for a fully working instance of sentry.

## Installing Docker

To install docker on RHEL 7 follow the steps below:

1. Ensure you have sudo or root provilidges on your machine.
2. Update all existing yum packages.
```
$ sudo yum update
```
3. Add the Docker Yum repo.
```
$ sudo tee /etc/yum.repos.d/docker.repo <<-EOF
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
```
4. Install the Docker Package.
```
$ sudo yum install docker-engine
```
5. Start the Docker deamon.
```
$ sudo service docker start
```
6. Verify `docker` is installed by running the `Hello World` test image in a container
```
$ sudo docker run hello-world
```
You should see the message 'Hello **from** Docker.'

7. Ensure the docker deamon starts at boot up
```
$ sudo chkconfig docker on
```

## Installing Docker Compose

We will use docker compose to create and manage all our docker containers that will run sentry from a single file. To install docker compose follow the instructions below:

1. Use curl to pull in to pull in the docker compose binary.
```
$ curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```
2. Make the binary executable
```
$ chmod +x /usr/local/bin/docker-compose
```
3. Check that the installation was successfull by running the following:
```
$ docker-compose --verison
```

## Building the Sentry containers

Now we have docker and compose installwe can build the containers required for runnign Sentry.

To simplify this process, I have created a repository that provides the recommended containers and config files. This has been forked from the official repo `getsentry/onpremise` with a couple of adjustments including setting up an nginx proxy server to access the sentry isntance. To start this process, clone the repo using `git` into a sentry directory.

```
$ git clone https://github.com/getsentry/onpremise.git sentry && cd sentry
```

This repository provides some default config files: `config.yml` and `sentry.conf.py`. More info on how these files can be customised is available at [Configuring Sentry](https://docs.sentry.io/on-premise/server/config/)

This can be customised as desired so I will leave this customisation out of the process.

In order to correctly set up your sentry virtual host in nginx make sure you replace `sentry.example.co.uk` in `nginx/conf/site.conf` with the host name you intend to access sentry through.

Now we must generate a secret key that will be used by sentry. Do this by running:

```
$ docker-compose run web config generate-secret-key
```

This will create the images in the docker compose file and generate a secret key which will be outputted to the console. The first time this is run on a machine docker will have to download all the images required. This will take a few minutes depending on your internet connection.

Now we have a randomly generated secret key we need to set it as a environment variable so sentry can access it. To do this open the `docker-compose.yml` file and uncomment the line with `SENTRY_SECRET_KEY: ''` under `services.base.environment` and place the secret key within the sungle quotes.

With the secret key set up we can now create a new user for the system. To do this run the following:
```
$ docker-compose run web upgrade
```
Complete the command promts to set up a new user.

Finally we can boot up the sentry service by running:
```
docker-compose up -d
```

Sentry will now be accessible on the host name specified earlier.
