---
layout: post
title: Docker Compose for nginx, PHP, Redis, and MySQL
comments: true
---

A friend of mine has a side project, currently deployed on AWS, using
[nginx][nginx] for static assets, and [PHP][php] for for the backend.
The backend connects to [Redis][redis] and [MySQL][mysql]. This is all
working fine, but he's had to set it up a couple times, and each time he
forgets some setting, and scratched his head for a while until he
remebers what was missed.

As well, he has been talking about migrating away from AWS to a dedicated server.

I've been talking up [Docker][docker], and
[Infrastructure As Code][iac]/[DevOps][devops] to him, so he asked me to show him.
This is what I came up with.

In addition to what I mentioned above, I'll be using [Docker Compose][docker-compose]
and [Traefik][traefik] in the following. You can find my final files on
[GitHub][github]

Ok, So I started with the following `docker-compose.yml` file:

    version: "3.7"

    services:
      proxy:
        image: traefik:v2.2
        command:
          - --api.insecure=true
          - --providers.docker
        ports:
          - 80:80
          - 443:443
          - 8080:8080
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock

Start Traefik by running `docker-compose up`, then visit
[http://localhost:8080][dashboard] and you should see the Traefik
dashboard. Press `ctrl-c` in the shell where you ran docker to
shout things down, or go to a separate shell, in the same directory,
and type `docker-compose down`.

Next, I added Nginx:

    web:
        image: nginx:latest
        labels:
          - "traefik.enable=true"
          - "traefik.http.routers.web.rule=Host(`web.docker.localhost`)"

If you refresh the [dashboad][dashboard], you should see an entry for
`web.docker.localhost`. If you visit [http://web.docker.localhost][webdocker],
you should see the nginx welcome page.

Then I created a local directory, called `nginx`. Inside it, i created anther directory,
`public`, and a file called `site.conf`:

    server {
      index index.php index.html;
      server_name web.docker.localhost;
      error_log  /var/log/nginx/error.log;
      access_log /var/log/nginx/access.log;
      root /public;
    }

Inside `public`, i created `index.html`:

    Hello Frustrated.Blog

I updated the `web` section of `docker-compose.yml`:

    web:
      image: nginx:latest
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.web.rule=Host(`web.docker.localhost`)"
      volumes:
        - ./nginx/public:/public
        - ./nginx/site.conf:/etc/nginx/conf.d/site.conf

Restart everything, and vist [web.docker.localhost][webdocker].
You should see `Hello Frustrated.Blog`.

Then, I added PHP:

    php:
      image: php:7.4-fpm
      volumes:
        - ./nginx/public:/public

Added `index.php` to `nginx/public`:

    <?php
    echo phpinfo();
    ?>

and added this to `nginx/site.cnf`:

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php_php_1:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

Restart everything, and load [web.docker.localhost][webdocker]. You should
see the PNP info page.

Time for Redis! Now, this requires installing a plugin to the PHP container.
My firend used [compose][compose] to install [predis][predis], so I needed
to do the same. After a lot of searching, I came up with this solution.

I created a `php` direcotry. Inside, I created `composer.json` which will
be used to install predis:

    {
      "require": {
          "predis/predis": "^1.1"
      }
    }

I also added a `Dockerfile` to build on top of `php:fpm-alpine`, install
composer, and then run composer to install predis.

    FROM php:fpm-alpine

    RUN docker-php-ext-install mysqli
    RUN curl -sS https://getcomposer.org/installer | php

    COPY composer.json ./
    RUN php composer.phar install

    CMD ["php-fpm"]

    EXPOSE 9000

And updated the `php` section of `docker-compose.yml`:

    php:
      build:
        context: php
        dockerfile: Dockerfile
      volumes:
        - ./nginx/public:/public

Then I added Redis:

    redis:
      image: redis:latest

I renamed `nginx/public/index.php` to `nginx/public/info.php`, and added
`nginx/public/redis.php`(this file was donated by my friend, as it has
been a long time since I wrote any PHP):

    <?php
      require_once '/var/www/html/vendor/predis/predis/autoload.php';

      $redis = new Predis\Client(['host' => 'redis']);
      $redisStatus = redisConnect($redis);

      // check redis I/O
      if ($redisStatus == "OK") {
          $ok = $redis->set("testKey", "testValue");
          if ($ok == "OK") {
              $okBool = $redis->get("testKey");
              if ($okBool) {
                  $redis->del("testKey");
                  $redisStatus = "Yessir. Readin' and ritin' to redis";
              } else {
                  $redisStatus = "Failed reading redis testKey";
              }
          } else {
              $redisStatus = "Failed writing redis testKey";
          }
      }

      // associative array maps nicely to json
      $payload = array(
        "greeting" => " Hello Moz!",
        "redis" => $redisStatus
      );

      // send a repsonse to caller
      sendResponse(200, json_encode($payload));

      // bye -- no close for redis
      exit(1);

      function sendResponse($status = 200, $body = '', $content_type = 'application/json')
      {
        // old-school http response
        $status_header = 'HTTP/1.1 ' . $status . ' OK';
        header($status_header);
        header('Content-type: ' . $content_type);
        header('Access-Control-Allow-Origin: *');
        header('Access-Control-Allow-Methods: GET, POST, OPTIONS, HEAD');
        header('Access-Control-Allow-Headers: apikey, Authorization, Origin, X-Requested-With, Content-Type, Accept');
        echo $body;
      }

      function redisConnect($mem) {
        try {
            $mem->connect();
            $mem->select(0);
            $status = "OK";
        }

        catch (Exception $exception) {
            $status = "Redis failed to connect";
        }
        return $status;
      }
    ?>


If you restarted the containers after the previous step, you'll need to purge
the old php container before you restart. The easieast way is to run

    docker container prune
    docker image prune -a

But remember, this will remove a lot of images! Maybe more than you want. The
other option is to use `docker container ls` and `docker image ls` to find the
image and container to remove.

Ok, restart everything, and visit [http://web.docker.localhost/redis.php][redisphp].
If everything worked, you should see:

    {"greeting":" Hello Moz!","redis":"Yessir. Readin' and ritin' to redis"}

Almost there! Only MySQL to add now.

I added `mysql` to `docker-compose.yml':

    mysql:
      image: mysql/mysql-server
      command: --default-authentication-plugin=mysql_native_password
      environment:
        MYSQL_ROOT_PASSWORD: password

And copied `redis.php` to `all.php`, and added the following:

    // check mysql I/O
    if ($mysqlStatus == "OK") {
      $data = [];
      $sql = "SELECT variable FROM sys_config;";
      $db = $mysql->query($sql);
      if ($db) {
          while($row = $db->fetch_assoc()) {
              $data[] = $row;
          }
          if (sizeof($data) < 1) {
              $mysqlStatus = "No data found";
          } else {
              $mysqlStatus = "All good. Found data," . sizeof($data) . " items";
          }
          $db->free();
      } else {
          $mysqlStatus = "Mysql query failed";
      }
    }

Restart everything, and load [all.php][allphp]. And, thereis an error.
After a lot of searching, I finally found a solution. MySQL in Docker will run
initialization scripts if ot starts and there are not databases.

So, I created a `mysql` directory, and inside created a
`docker-entrypoint-initdb.d` directory. In there, I created `script.sql`:

    use mysql;
    CREATE USER 'root'@'%' IDENTIFIED BY 'root';
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

and modified `docker-compose.yml`:

Note: YOu will need to remove the mysql container before restarting, to
force the config scripts to run.

Load [all.php][allphp], and you should see:

    {"greeting":" Hello Moz!","redis":"Yessir. Readin' and ritin' to redis","mysql":"All good. Found data,6 items"}

We did it! nginx, PHP, Redis, and MySQL! All started with one command!

Hopefully this saves some one else from the hours of frustration I expereinced
getting to this point!


[nginx]: https://nginx.org/
[php]: https://www.php.net/
[redis]: https://redis.io/
[mysql]: https://www.mysql.com/
[docker]: https://www.docker.com/
[iac]: ...
[devops]: ...
[docker-compose]: ...
[traefik]: ...
[github]: https://github.com/abendigo/nginx-php-redis-mysql
[dashboard]: http://localhost:8080
[webdocker]: http://web.docker.localhost
[redisphp]: http://web.docker.localhost/redis.php
[allphp]: http://web.docker.localhost/all.php
