
# Elixir/Phoenix Development Environment

## Using This Environment

This environment is simple to use. Clone the environment to your host computer.
Edit the Dockerfile to select the version of Elixir and Phoenix that you want to
use. Build the image with

    $ docker-compose build

In order to run mix commands alias the containers mix command with

    $ alias mix="docker-compose run --rm phoenix mix"

Now mix being sent into the container. So for example, you could use it to create a
new project by running

    $ mix phx.new . --app hello

This will create an application called hello in the /app directory and copy the
code into the /src directory on the host. Once that is done simply edit the
src/config/dev.ex from

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
        hostname: "db",         ### to this
        show_sensitive_data_on_connection_error: true,
        pool_size: 10

and run 

    $ mix ecto.create

to create the database. Data will be saved into the the volume /pgdata on the
host. Finally run either

    $ docker-compose up

or 

    $ docker-compose up -d

for detached mode.

You can get a teminal in the container by running

    $ docker exec -it <continer_id | container_name>

Simply send any mix command to the container by typing

    $ mix command [args]

at the host command prompt. You can edit the project by running your favorite
code editor in the host, and you can view your project at localhost:4000 in your
host browser.