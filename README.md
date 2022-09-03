
# Elixir/Phoenix Development Environment

## Yeah, but Why?

I created this environment so that I could have a self contained environment to
use, from development through production, and not have to worry about conflicts
between different versions of software on my host and/or production environment.

## How It Works

The environment uses a Dockerfile along with a docker-compose.yml file to create an
environment that lets you use Docker to generate a Docker container and run the
container on the host OS. Once the container is up and running you cna generate a
project with the container that is written into a volume on the host using mix
from the host terminal as normaly would. The environment makes it easy to make
changes to the code in the host volume as you normally would . You can then view
the project on the host browser at localhost:4000. In short, once the container
is built your workflow is the same as if you were working in a project built and
running on the host with the execpetion of starting the server, which uses the
`docker-compose up` command. But, I'll get to that in a bit.

## Initializing The Environment

After pulling down the environment by either forking or cloning it in to a
folder on your local computer there are a few steps you need to take to get it
ready for use. First, in order to make it easy to work with the environment I
used a `.env` file in the `.dev/` directcory to set up an alias so that any mix
command you run in the terminal(s) you are using for development is executed
against the Elixir code running in the container. In order to set the alias run
the following command in the root of the project on the host.

`$ source .dev/.env`

Next, you need to build the docker image that will create your container. As
previously mentioned both the Dockerfile and docker-compose.yml are in the
`.dev/` directory. This means to build them you either need to `cd` into the
`dev/` directory or run the following command from the root of the project.

`$ docker-compose -f .dev/docker-compose.yml`

## Developing An App

Now, in order to develope your application the work flow is nearly identical to
working on an app generated on your localhost. You use mix commands in the same
way as you normally would with the execption of starting the server.
Additionally, you need to make two changes in the `config/dev.ex`. Change the
database host name from `localhost` to `db` and change your ip from `http: [ip: 
{127, 0, 0, 1}, port: 4000]` to `http: [ip: {0, 0, 0, 0}, port: 4000]`. Finnally
you will need to change the owner of the `src/` dirctory from `root:root` to
`user:user` where user is your username. Lets do that now by running the
following commands in the root of the poject on localhost. (Note: Remember to
update the `config/dev.ex` or you will get an error when you try and generate
your db.)

1. Generate The Project

`$ mix phx.new . --app <application_name> [ options ]`

2. Change The Owner of src/ on Host

`$ sudo chown -R user:user src/`

3. `$ mix ecto.create`

`$ docker-compose -f .dev/docker-compose.yml up`

## Stoping and Starting The Container

To run your sever you need to run docker-compose up against the container.
Similarly to stop the server you need to bring the docker container with
docker-compose down. One thing that is a little bit surprising is that if you
decide to remove the `pgdata/` directory for any reason you need to stop the
container first because otherwise the container running the db service will
simply it. So, to stop and start the containers this project uses you simply run
the following commands.

__Start the Container__

`$ docker-compose -f .dev/docker-compose.yml up`

__Stop the Container__

`$ docker-compose -f .dev/docker-compose.yml down`

If you want to re









## Using This Environment

This environment is simple to use. Fork or clone the environment into
`my_project` on your host computer. Add a `my_project/src/` directory for the docker container
to store your project in. Edit the Dockerfile to select the version of Elixir
and Phoenix that you want to use for your project and build the image with the
following command. 

    $ docker-compose build

This image uses the image `elixier:latest`, but it's best
practice to choose a specific version of software rather than using latest to
insure your project is not effected by breaking changes.

Run `$ source .dev/.env` to set up an alias so that the mix command runs
aginst the container instead of on your host. Now that mix comands from this
terminal are running in container you can create a new project with

    $ mix phx.new . --app <application_name>

This will create an application called `application_name` in the containers /app
directory and copy the code into the `my_project/src` directory on the host.
Once that is done You can simply edit the src/ code on the host and see changes
reflected on the webpage running at `localhost:4000` on the host browser.

In order to create a database you need to edit the src/config/dev.ex by changing
`hostname: "localhost` to `hostname: "db` which is the database service
referenced in the docker-compose.yml file.

    config :hello, Hello.Repo,
        username: "postgres",
        password: "postgres",
        database: "hello_dev",
        hostname: "localhost",   ### change this line 
        show_sensitive_data_on_connection_error: true,
        pool_size: 10

to

    config :hello, Hello.Repo,
        username: "postgres",
        password: "postgres",
        database: "hello_dev",
        hostname: "db",         ### to this line 
        show_sensitive_data_on_connection_error: true,
        pool_size: 10

and run 

    $ mix ecto.create

Data will be written into the the volume `/pgdata` on the host so it will be
available the container is subsequently run. Finally run either

    $ docker-compose up

or 

    $ docker-compose up -d

for detached mode.

You can get a teminal in the container by running

    $ docker exec -it <continer_id | container_name>

Again, you send any mix command to the container by typing

    $ mix command [args]

at the host command prompt just as you would if you were developing on the host
machine.

Note: In order to work there must be `my_project/src/` directory in the project on the host
computer, and this directory must be created from the host command line.
