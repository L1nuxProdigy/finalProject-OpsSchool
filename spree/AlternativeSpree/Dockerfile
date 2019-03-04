FROM ruby:2.4.1

MAINTAINER hilavalensia@gmail.com

RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs ruby-dev imagemagick
RUN apt-get update

ENV APP_HOME /app
RUN mkdir $APP_HOME
WORKDIR $APP_HOME

ADD Gemfile* $APP_HOME/
RUN bundle install

ADD . $APP_HOME

RUN rails db:migrate

RUN rake test

CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]
