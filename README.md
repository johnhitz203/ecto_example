
# Elixir/Phoenix Development Environment

## Why I Created This Environment

I created this environment so that I could have a self contained environment to
use, from development through production, and not have to worry about conflicts
between different versions of software on my host and/or production environment.

## How It Works

The environment uses a `.dev/Dockerfile` along with a `.dev/docker-compose.yml`
file to create an environment that lets you use Docker to generate a Docker
container and run the container on the host OS. Once the container is up and
running you can generate a project inside the container that is also written
into a volume on the host using mix as you normaly would. The volume is writen
into the `src/` directory in the root of the poject so it is as easy to make
changes as it is in a normal mix project. You can also view the project on the
host browser at localhost:4000. In short, once the container is built your
workflow is the same as if you were working in a project built and running on
the host with the execpetion of starting the server, which uses the
`docker-compose up` command. But, I'll get to that in a bit.

## Initializing The Environment

After pulling down the environment by either forking or cloning it into a
folder on your local computer there are a few steps you need to take to get it
ready for use. First, you can remove the `src/` directory which contains a very
simple example application. Next you need to `source` the `.dev/.env` file in
order to created alias' for running the commands that execute the functionality
of the environment. After sourcing the `.dev/.env` file any mix mix command you
run in the development terminal are executed against the Elixir code running in
the container. (This needs to be done for all terminals you are using for
development.) After that you need to build the docker image that will create
your container. 

`$ rm -rf src/`

`$ source .dev/.env`

`$ build`

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