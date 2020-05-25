# Installing

Response is super simple to install and setup. If you haven't checked out the platform requirements, please
make sure to [check out those requirements here before continuing.](./02-Requirements.md).

<!-- theme: info -->

> ### Hosted Response Coming Soon
>
> Response will soon be available as a paid hosted option which will allow you to have your own dedicated Response instance
> without having to manage, backup, and update your server. In addition to making life easier, Hosted Response will help fund
> the development of future Respsnse features and give you access to our support portal for free.
>
> This is an **optional service** and is not required to use Response, which is MIT-licensed. Consider supporting us if you
> can.

## Prerequisites

1. Response is based on the Laravel framework and as such **requires PHP**. We officially support **PHP 7.2+**,
   you'll need this in order to pass the system requirements check in the installer.

2. Response utilizes Composer for dependency management, you'll need [Composer installed](https://getcomposer.org/doc/00-intro.md)
   before installing Response.

3. You'll need a web server that matches our [server requirements](./02-Requirements.md). For local developemt you can use
   [Laravel Homestead](https://laravel.com/docs/7.x/homestead) or [Laravel Valet](https://laravel.com/docs/7.x/valet). Both
   of these will meet or exceed the system requirements for the Response application.

4. You'll need `MySQL 8+` or `MariaDB 10.4+`.

5. You'll need Nginx or Apache. We recommend using Nginx.

## Recommendations

### Queuing

To keep the user interface snappy Response uses background tasks to perform long-running actions such as sending email, installing
Composer packages, taking backups, etc. Background queue processing is done synchronously by default which will cause long loading
times. It is highly recommended that you configure Redis for queueing.

Instructions for installing and configuring Redis are available in this documentation.

### Caching

Response caches database queries, settings, and other things to keep load times to a minimal. Response is configured to use the local
filesystem for the cache source by default. This will work fine in development environments and smaller production environments where
there are fewer people using Response. For larger installations or many simultaneous active users we recommend configuring Redis for
caching as well.

<!-- theme: warning -->

> #### Using Redis for Caching and Queing
>
> If you use Redis for both caching and queuing it is recommended that you use separate redis databases (not servers)
> for each functionality. This will help with both performance as well as provides a logical separation of the data should
> you ever need to inspect it.

## Get Response

Response is available through various download methods.

### Composer (Recommended)

Since Composer should already be installed on your server per the [prerequisites listed above](#prerequisites), we recommend the use
of Composer's `create-project` command for the simplest installation steps.

```shell
# Replace `my-responce` with the directory you'd like to install Resonse in.
composer create-project --prefer-dist responseams/response my-response
```

### Git

Response can be downloaded as a GitHub release from [the releases section](https://github.com/responseams/response/releases) fo the
Response project repository. You might also wish to clone the repository if you have Git installed.

<!--
type: tab
title: Cloning with SSH
-->

```shell
# Replace `my-response` with the directory you'd like to clone Response into.

# Replace {version} with the Response version you intend to use.
git clone git@github.com:responseams/response.git --depth 1 --single-branch --branch {version} my-response

# Or fetch the master branch for the latest changes that may not be released.
git clone git@github.com:responseams/response.git --depth 1 --single-branch --branch master my-response
```

<!--
type: tab
title: Cloning with HTTPS
-->

```shell
# Replace `my-response` with the directory you'd like to clone Response into.

# Replace {version} with the Response version you intend to use.
git clone https://github.com/responseams/response.git --depth 1 --single-branch --branch {version} my-response

# Or fetch the master branch for the latest changes that may not be released.
git clone https://github.com/responseams/response.git --depth 1 --single-branch --branch master my-response
```

<!-- type: tab-end -->

## Directory Permissions

Once Response has been fetched and is available in a local folder you may need to configure some permissions for the application
to function correctly. The `storage` and `bootstrap/cache` directories should be writable by your web server or Response will not
run. If you are using [Laravel Homestead](https://laravel.com/docs/7.x/homestead) or [Valet](https://laravel.com/docs/7.x/valet)
this will already be configured properly for you.

To configure permissions, use the following commands. The following commands assume that `/path/to/response` is the full path to
the Response files on your server and that `www-data` is the name of the group that your webserver's user is a member of.

1. Give yourself and the `www-data` group access to the files.

```shell
sudo chown -R my-user:www-data /path/to/response
```

2. Set all directory permissions to `755`

```shell
sudo find /path/to/response -type d -exec chmod 755 {} \;
```

3. Set all file permissions to `644`

```shell
sudo find /path/to/response -type f -exec chmod 644 {} \;
```

4. Set permissions for the `storage` and `bootstrap/cache` directories

```shell
sudo chgrp -R www-data storage bootstrap/cache
sudo chmod -R ug+rwx storage bootstrap/cache
```

## Web Server Configuration

Your web server should be configured to only make the `public` directory available to the web. The `index.php` file within this
directory will server as an entrypoint for all routes, it is not necessary for you to make any changes to this directory, all
changes will be made automatically. You should, however, ensure that your document/web root is set to the `public` directory
of Response.

Response does not currently support being installed in sub-directories. Please only install Response at the root. For example,
installing Response at `https://example.com/response` is not supported, while `https://example.com` or `https://response.example.com`
are both aceptable.

## Pretty URLs

### Apache

As mentioned before, Response includes a `.htaccess` file that is used to provide URLs without the `index.php` in the path. Before
serving Response with Apache, be sure to enable the `mod_rewrite` module so that the `.htaccess` file will be honored by Apache.

If the `.htaccess` file included in Response's public folder doesn't work for you, try this alternative:

<!--
title: "Alternate Apache .htaccess configuration"
-->

```apache
Options +FollowSymLinks -Indexes
RewriteEngine On

RewriteCond %{HTTP:Authorization} .
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```

### Nginx

If you are using Nginx (which we recommend), the following configuration adopted from the Laravel documentation should work nicely.
You will likely need to make some tweaks to match your directory structure. The following Nginx configuration assumes you are using
PHP FastCGI Process Manager (PHP-FPM). Our complete installation guide covers configuring this for an optimal performing Response
instance.

<!--
title: "Nginx configuration"
-->

```nginx
server {
    listen 80;
    server_name response.example.com;
    root /response.example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

## Installing Composer Dependencies

If you are installing Response for **production use** and don't intend to develop themes, modules, or make changes to the included
Laravel application, use the following command to optimize the Composer autoloader.

```shell
composer install --optimize-autoloader --no-dev
```

If you do intend to develop themes or modules for Response, simply install Composer dependencies for development as well as production.
We do not recommend that you host a development version of Response for others to use. This should really only be used if you are using Larvel Homestead, Valet, or locally developing on Response.

```shell
composer install
```

## Run the Response Installer

Once you have configured your server, navigating to your Response instance using the configured domain or IP address will show you a
page indicating that Response is not yet ready and you must run the Response Installer in the command line. If you see this page, it's
safe to run the Response Installer in the command line. Make sure you are in the Response directory in your terminal before running the following command.

```shell
php artisan response:install
```

When ran without any parameters you will be prompted to configure everything using a prompt-based installer that will ask you questions
to configure Response to connect to your database, populate the database, setup encryption keys, and a number of other tasks.

You can follow the prompts to install Response in production mode. If you are using Laravel Homestead, providing `--homestead` to the command
will automatically configure Response to work with Homestead. Using `--valet` will configure Response to use your local database, but you'll
still need to configre a few other settings which you will be prompted for.

When configuring Response without `--valet` or `--homestead` you will be configuring Response in production mode. To force debug mode or
development mode add the `--debug` or `--dev` flags respectively.

Response will by default use `localhost` and port `3306` for your MySQL/MariaDB server. You can change these using the command line flags
`--db-host` and `--db-port` respectively. Other options can be configured via command line flags as well, see the entire output from the help command below.

You can run the same command to see it locally as well.

```shell
$ php artisan response:install --help

Description:
  Perform the installation tasks for Response.

Usage:
  response:install [options]

Options:
  -f, --force                        Forces an installation if Response is already installed.
  -r, --refresh                      Forces a reinstallation of Response, retaining modules, themes, and datasets.
      --homestead
      --valet
      --dev
      --debug
  -c, --show-config                  Dumps the final config, useful for debugging.
  -u, --url[=URL]                    The URL Response will be hosted at.
      --admin-name[=ADMIN-NAME]       [default: "Default Admin"]
      --admin-email[=ADMIN-EMAIL]     [default: "admin@example.com"]
      --db-host[=DB-HOST]             [default: "localhost"]
      --db-port[=DB-PORT]             [default: "3306"]
      --db-database[=DB-DATABASE]
      --db-username[=DB-USERNAME]
      --db-password[=DB-PASSWORD]
      --db-prefix[=DB-PREFIX]
      --db-charset[=DB-CHARSET]       [default: "utf8mb4"]
      --db-collation[=DB-COLLATION]   [default: "utf8mb4_unicode_ci"]
  -h, --help                         Display this help message
  -q, --quiet                        Do not output any message
  -V, --version                      Display this application version
      --ansi                         Force ANSI output
      --no-ansi                      Disable ANSI output
  -n, --no-interaction               Do not ask any interactive question
      --env[=ENV]                    The environment the command should run under
  -v|vv|vvv, --verbose               Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
```

A default admin account is created using `admin@example.com` as the email and `Default Admin` as the name. You can change those as shown above, however a randomly
generated password will always be provided. You will see this password as well as all of the credentials needed to login as the default admin account once Response has finished installing.

## Other Questions

We're happy to help you get things going if you have questions, just reach out on Discord and if you hit a bug be sure to report it in the GitHub
issue tracker at [https://github.com/responseams/response/issues](https://github.com/responseams/response/issues).
