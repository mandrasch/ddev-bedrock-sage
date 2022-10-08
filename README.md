# DDEV Bedrock + Sage example

- [DDEV](https://ddev.readthedocs.io/en/latest/)
- [Roots Bedrock](https://roots.io/bedrock/)
    - [DDEV Quickstart for Bedrock](https://ddev.readthedocs.io/en/latest/users/quickstart/#bedrock)
- [Roots Sage Starter Theme](https://github.com/roots/sage)

🚧 Work in progress 🚧

## Setup for local development:

```bash
ddev start 
ddev composer install

# Set .env values for local DDEV environment
# (via https://github.com/aaemnnosttv/wp-cli-dotenv-command)
ddev wp package install aaemnnosttv/wp-cli-dotenv-command:^2.0
ddev wp dotenv init --with-salts --force
ddev wp dotenv set DB_HOST 'db' &&\
    ddev wp dotenv set DB_NAME 'db' &&\
    ddev wp dotenv set DB_USER 'db' &&\
    ddev wp dotenv set DB_PASSWORD 'db' &&\
    ddev wp dotenv set WP_HOME "$\{DDEV_PRIMARY_URL}" &&\
    ddev wp dotenv set WP_SITEURL "$\{WP_HOME}/wp" &&\
    ddev wp dotenv set WP_ENV 'development'

# Finish installation in browser:
ddev launch

# Activate sage theme afterwards:
ddev wp theme activate my-sage-theme

# Jump into DDEV docker container:
ddev ssh
cd web/app/themes/my-sage-theme/
composer install
yarn install

# run 'bud dev' and watch for changes:
yarn dev
```

## How was this created?

```bash
# Roots bedrock
# https://ddev.readthedocs.io/en/latest/users/quickstart/#bedrock
ddev config --project-type=wordpress --docroot=web --create-docroot
ddev start
ddev composer create roots/bedrock

# Set .env values for DDEV
# via https://github.com/aaemnnosttv/wp-cli-dotenv-command
ddev wp package install aaemnnosttv/wp-cli-dotenv-command:^2.0
ddev wp dotenv init --with-salts --force
ddev wp dotenv set DB_HOST 'db' &&\
    ddev wp dotenv set DB_NAME 'db' &&\
    ddev wp dotenv set DB_USER 'db' &&\
    ddev wp dotenv set DB_PASSWORD 'db' &&\
    ddev wp dotenv set WP_HOME "$\{DDEV_PRIMARY_URL}" &&\
    ddev wp dotenv set WP_SITEURL "$\{WP_HOME}/wp" &&\
    ddev wp dotenv set WP_ENV 'development'

# Finish installation via browser:
ddev launch

# Roots sage
# https://docs.roots.io/sage/10.x/installation/
ddev composer require roots/acorn
# Change into DDEV container, because we need 
# to run composer create-project in themes directory
ddev ssh
cd web/app/themes/
composer create-project roots/sage my-sage-theme
cd my-sage-theme
# test build:
yarn
exit 

# Activate theme
ddev wp theme activate my-sage-theme
ddev launch
```

Add browsersync support for HTTPS, fork of https://github.com/drud/ddev-browsersync/blob/main/docker-compose.browsersync.yaml:

```yaml
# .ddev/docker-compose.browsersync.yaml
# Override the web container's standard HTTP_EXPOSE and HTTPS_EXPOSE
# This is to expose the browsersync port.
version: '3.6'
services:
  web:
    expose:
      - '3000'
    environment:
      - HTTP_EXPOSE=${DDEV_ROUTER_HTTP_PORT}:80,${DDEV_MAILHOG_PORT}:8025,3001:3000
      - HTTPS_EXPOSE=${DDEV_ROUTER_HTTPS_PORT}:80,${DDEV_MAILHOG_HTTPS_PORT}:8025,3000:3000
```
See: https://github.com/drud/ddev-browsersync/pull/21

Restart required via `ddev restart`!

Also we need to modify `bud.config.mjs`:

```javascript
    /**
     * Proxy origin (`WP_HOME`)
     */
    // .proxy("http://example.test")
    .proxy('https://ddev-bedrock-sage.ddev.site')
    /**
     * Development origin
     */
    // .serve("http://0.0.0.0:3000")
    .serve('https://ddev-bedrock-sage.ddev.site:3000')
```

See: https://discourse.roots.io/t/sage-10-ddev-browsersync-exposing-port-3000-400-bad-request/22215/2

