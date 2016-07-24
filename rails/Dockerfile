FROM ruby:2.1

RUN apt-get update -qq && \
    apt-get install -y libpq-dev nodejs && \
    apt-get clean && \
    apt-get autoclean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
    
ENV APP_HOME /application
RUN mkdir $APP_HOME
WORKDIR $APP_HOME

ADD Gemfile* $APP_HOME/
RUN bundle install --without test development 

ADD . $APP_HOME
ADD config/database.docker.yml $APP_HOME/config/database.yml

ENV DATABASE_NAME orienteer
ENV THREADS 1:20
ENV WORKERS 2

ENV PORT 3006
EXPOSE 3006

CMD bash -c 'puma -t $THREADS -w $WORKERS --preload -b tcp://0.0.0.0:$PORT'
