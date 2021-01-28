# Blazer Docker

A Docker image for [Blazer](https://github.com/ankane/blazer)

## Getting Started

Pull the image

```sh
docker pull ankane/blazer
```

Create database tables

```sh
docker run -ti -e DATABASE_URL=postgres://user:password@hostname:5432/dbname ankane/blazer rails db:migrate
```

Run the web server

```sh
docker run -ti -e DATABASE_URL=postgres://user:password@hostname:5432/dbname -p 8080:8080 ankane/blazer
```

And visit [http://localhost:8080](http://localhost:8080)

> On Mac, use `host.docker.internal` instead of `localhost` to access the host machine (requires Docker 18.03+)

## Authentication

Add basic authentication with:

```sh
-e BLAZER_USERNAME=admin -e BLAZER_PASSWORD=secret
```

Or use a reverse proxy like [OAuth2 Proxy](https://github.com/oauth2-proxy/oauth2-proxy).

## Checks

Schedule checks to run (with cron, [Heroku Scheduler](https://elements.heroku.com/addons/scheduler), etc). The default options are every 5 minutes, 1 hour, or 1 day, which you can customize. For each of these options, set up a task to run.

```sh
docker run ... rake blazer:run_checks SCHEDULE="5 minutes"
docker run ... rake blazer:run_checks SCHEDULE="1 hour"
docker run ... rake blazer:run_checks SCHEDULE="1 day"
```

You can also set up failing checks to be sent once a day (or whatever you prefer).

```sh
docker run ... rake blazer:send_failing_checks
```

For Slack notifications, create an [incoming webhook](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) and set:

```sh
-e BLAZER_SLACK_WEBHOOK_URL=https://hooks.slack.com/...
```

Name the webhook “Blazer” and add a cool icon.

### Mailing

Create a `mailer.rb` file with:

```ruby
Rails.application.config.action_mailer.default_url_options = {
  host: ENV['HOST'],
}
Rails.application.config.action_mailer.delivery_method = :smtp
Rails.application.config.action_mailer.smtp_settings = {
  authentication: :plain,
  address: ENV['MAIL_SMTP_URL'],
  port: ENV['MAIL_SMTP_PORT'],
  user_name: ENV['MAIL_SMTP_USERNAME'],
  password: ENV['MAIL_SMTP_PASSWORD'],
  domain: ENV['MAIL_SMTP_DOMAIN']
}
Rails.application.config.action_mailer.default_options = {
  from: ENV['MAIL_SMTP_FROM']
}
```

Copy the `mailer.rb` file into your image (using the `Dockerfile`) and build it (like in Customization):

`COPY mailer.rb /app/config/initializers/mailer.rb`

Make sure to populate all environment variables for runtime.

## Customization

Create a `blazer.yml` file with:

```yml
# see https://github.com/ankane/blazer for more info

data_sources:
  main:
    url: <%= ENV["DATABASE_URL"] %>

    # statement timeout, in seconds
    # none by default
    # timeout: 15

    # caching settings
    # can greatly improve speed
    # off by default
    # cache:
    #   mode: slow # or all
    #   expires_in: 60 # min
    #   slow_threshold: 15 # sec, only used in slow mode

    # wrap queries in a transaction for safety
    # not necessary if you use a read-only user
    # true by default
    # use_transaction: false

    smart_variables:
      # zone_id: "SELECT id, name FROM zones ORDER BY name ASC"
      # period: ["day", "week", "month"]
      # status: {0: "Active", 1: "Archived"}

    linked_columns:
      # user_id: "/admin/users/{value}"

    smart_columns:
      # user_id: "SELECT id, name FROM users WHERE id IN {value}"

# create audits
audit: true

# change the time zone
# time_zone: "Pacific Time (US & Canada)"

# email to send checks from
# from_email: blazer@example.org

# webhook for Slack
# slack_webhook_url: <%= ENV["BLAZER_SLACK_WEBHOOK_URL"] %>

check_schedules:
  - "1 day"
  - "1 hour"
  - "5 minutes"

# enable map
# mapbox_access_token: <%= ENV["MAPBOX_ACCESS_TOKEN"] %>
```

Create a `Dockerfile` with:

```Dockerfile
FROM ankane/blazer

COPY blazer.yml /app/config/blazer.yml
```

And build your image:

```sh
docker build -t my-blazer .
```

## Deployment

### Health Checks

Use the `/health` endpoint for health checks. Status code `200` indicates healthy.

## History

View the [changelog](https://github.com/ankane/blazer-docker/blob/master/CHANGELOG.md)

## Contributing

Everyone is encouraged to help improve this project. Here are a few ways you can help:

- [Report bugs](https://github.com/ankane/blazer-docker/issues)
- Fix bugs and [submit pull requests](https://github.com/ankane/blazer-docker/pulls)
- Write, clarify, or fix documentation
- Suggest or add new features

To get started with development:

```sh
git clone https://github.com/ankane/blazer-docker.git
cd blazer-docker
docker build -t ankane/blazer:latest .
```
