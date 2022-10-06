FROM elixir:1.13
RUN adduser -u 1000 johnhitz

RUN apt-get update && \
    apt-get install -y postgresql-client && \
    apt-get install -y inotify-tools && \
    apt-get install -y nodejs && \
    curl -L https://npmjs.org/install.sh | sh && \
    curl -L https://fly.io/install.sh | sh
    
ENV PATH="${PATH}:/root/.fly/bin:root/.fly/bin"
ENV APP_HOME /app
RUN mkdir $APP_HOME

WORKDIR $APP_HOME

RUN chown johnhitz:johnhitz $APP_HOME
# USER johnhitz
RUN mix local.hex --force && \
    mix archive.install hex phx_new 1.6.11 --force && \
    mix local.rebar --force

ENTRYPOINT ["tail", "-f", "/dev/null"]
