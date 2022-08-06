
# Elixir/Phoenix Development Environment

## Using This Environment

This environment is simple to use. Clone the environment to your host computer
and add a `src/` directory for the docker container to store your project in.
Edit the Dockerfile to select the version of Elixir and Phoenix that you want to
use for your project and build the image with (It is best practice to choose a
specific version of software rather than using latest)

    $ docker-compose build

You can run commands against a container with `docker-compose run`. In
order to make it simple to run `mix` commands it is helpful to alias mix to

    $ alias mix="docker-compose run --rm phoenix mix"

Now mix is being sent into the container. Use it to create a new project with

    $ mix phx.new . --app <application_name>

This will create an application called `application_name` in the containers /app
directory and copy the code into the /src directory on the host. Once that is
done You can simply edit the src/ code on the host and see changes reflected on
the webpage running at `localhost:4000` on the host browser.

In order to create a database you need to edit the src/config/dev.ex by changing
`hostname: "localhost` to `hostname: "db` which is the database service which is
taken from the docker-compose.yml file.

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

Data will be written into the the volume `/pgdata` on the
host so it will be available when the subsequently run. Finally run either

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

Note: In order to work there must be `src/` directory in the project on the host
computer, and this directory must be created from the host command line.