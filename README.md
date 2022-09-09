
# Elixir/Phoenix Development Environment

## Why I Created This Environment

I created this environment so that I could have a self contained environment to
use, from development through production, and not have to worry about conflicts
between different versions of software on my host and/or production environment.

## How It Works

The environment uses `.dev/Dockerfile` along with `.dev/docker-compose.yml`
files to create an environment that lets you use Docker to generate a Docker
container and run the container on the host OS. Once the container is up and
running you can use `mix` as you normally would to generate a project inside the
container. The project is created in the `app/` directory in the container and
written to the coresponding volume located in the `src/` directory on the host.

Having the `src/` directory in the root of the poject makes it easy to make
changes. You simply edit the files and use mix commands as normal. Because the
container port 4000 is mapped to the host port 4000 you can also view the
project on the host browser at localhost:4000. In short, once the container is
built your workflow is the same as if you were working in a project built and
running on the host with the execpetion of starting the server, which uses the
`docker-compose up` command and deployment, which is done with an interactive
terminal in the Docker container. But, I'll get to those details in a bit.

## Initializing The Environment

After pulling down the environment by either forking or cloning it into a
folder on your local computer there are a few steps you need to take to get it
ready for use. First, you should remove the `src/` directory which contains a
very simple example application. After that you should change the `git remote`
to point to the git repository for the poject you're creating. Next you need to
run `$ source .dev/.env` in the host terminal file order to create the alias'
for running the commands that execute the functionality of the environment.
Sorcing the `.dev/.env` file makes it so any mix commands you run in the
development terminal will be executed against the Elixir code running in the
container. This step needs to be done for all terminals you are using for
development. Finally, you need to build the docker image that will create
your container. Here are the commands to do that.

```
$ rm -rf src/
$ git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
$ source .dev/.env
$ build
```

## Developing An App

Now, in order to develope your application the work flow is nearly identical to
working on an app generated on your localhost. You use mix commands in the same
way you normally would with the execption of starting the server. Additionally,
you will need to change the owner of the `src/` dirctory from `root:root` to
`user:user` where user is your username. Finally, you need to make two changes
in the `src/config/dev.ex`: change the database host name from `localhost` to `db`
and change your ip from `http: [ip: {127, 0, 0, 1}, port: 4000]` to `http: [ip:
{0, 0, 0, 0}, port: 4000]`. Lets do that now.

`$ mix phx.new . --app <application_name> [ options ]`

`$ sudo chown -R user:user src/`

`$ mix ecto.create`

`$ up`

## Stoping and Starting The Container

To run your sever you need to run docker-compose up against the container.
Similarly to stop the server you docker-compose down. One thing that can be a
little bit surprising is if you decide to remove the `pgdata/` directory
for any reason you need to stop the container first because otherwise the
container running the db service will simply recreate it. If you don't intend to
use the `db` service you can simply comment it out in the
`.dev/docker-compose.yml` So, to stop and start the containers used by this
project simply run the following commands. 

__Start the Container__

`$ up`

__Stop the Container__

`$ down`

## Deployment

__Deploy to fly.io__

The container that creates this environment includes the fly.io CLI. The
deployment process for fly.io is about as simple as it gets and there docs are
quite thorough so I will not receate them here. That said, you do need to run
the fly CLI command in an interactive terminal inside the container

This means you have to have the container id which you will use to open a
terminal. That can be obtained by running 

`$dps`

which should return something that looks like this.

```
CONTAINER ID   IMAGE          COMMAND                  CREATED             STATUS             PORTS                                       NAMES
ad578a95b7db   dev_phoenix    "mix phx.server"         About an hour ago   Up About an hour   0.0.0.0:4000->4000/tcp, :::4000->4000/tcp   dev_phoenix_1
7f1986c05eab   postgres:9.6   "docker-entrypoint.s…"   About an hour ago   Up About an hour   5432/tcp                                    dev_db_1
```
We are looking for the `CONTAINER ID` that coresponds with the `dev_phoenix`
image. Using that information open an interactive terminal in the container with

`$ it ad578a95b7db`

This will change your command prompt to something similar to `root@ad578a95b7db:/app#`. Fly.io uses
`fly launch` to initialially build and deploy the app and `fly deploy` to push
updates to the running application. But, before you can use those commands you
have to login by typing the following command.

`root@ad578a95b7db:/app# fly auth login`

You will be presented with a message similar to 

```
failed opening browser. Copy the url (https://fly.io/app/auth/cli/4490b6fe0c93e612af90738423bc7985) into a browser and continue
Opening https://fly.io/app/auth/cli/4490b6fe0c93e612af90738423bc7985 ...

Waiting for session...
```

Once you get the interactive terminal prompt `root@ad578a95b7db:/app#` back you
are ready to go.

`$