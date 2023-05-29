# dev_laravel
Base setup and instructions for setting up a docker stack for Laravel development

This directory contains settings files for both sublime text 3 and Visual Studio Code.
If using sublime, you can double-click on the project file. If using Visual Studio Code (VSCode), Open a new window and select File->Open Folder before navigating to the directory containing this file. The VSCode method includes recommended extensions and settings, which you can, of course, override as you choose, that are, after all my selections as the author of this particular sandbox. 

Also included in the stack, I have implemented xDebug for PHP, and three monitoring tools for the stack cadvisor, prometheus, and grafna which, I believe, make load testing easier to evaluate as most primary metrics are visible in an easily consumed way.

**FIRST**


To install a new Laravel instance for a project, use the composer container defined in docker-compose:

Change into the project base directory, then.

    find . -name '.DS_Store' -type f -delete
    find . -name '__MACOSX' -type d -delete

Ensure the target and database directories are empty

    rm -rf ./src/.* ./src/*
    rm -rf ./database/.* ./database/*

use composer to install Laravel into the current directory
  (To install a specific version of Laravel append the version number to the end of the following [i.e.  6.0.\*], or leave it for the current stable version)

    cd ./src
    docker-compose run --rm composer create-project laravel/laravel=^9.0 .

Edit DB setting in .env (Laravel environment file)

From:

    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=laravel
    DB_USERNAME=root
    DB_PASSWORD=

To:

    DB_CONNECTION=mysql
    DB_HOST=mysql
    DB_PORT=3306
    DB_DATABASE=laravel
    DB_USERNAME=root
    DB_PASSWORD='SdkW7BkXsKY6i82UuQ34'


and

From:

    REDIS_HOST=127.0.0.1
    REDIS_PASSWORD=null
    REDIS_PORT=6379
To:

    REDIS_HOST=redis
    REDIS_PASSWORD=null
    REDIS_PORT=6379

### Now build the rest of the stack

Return to the development base directory

    cd ../

Before building the remainder of the stack, create the storage directories as these are mounted are bound volumes so they need to exist before container creation.

    mkdir -p database/redis
    mkdir -p database/mongodb/db
    mkdir -p database/mongodb/dev.archive
    mkdir -p database/mongodb/production

Now run the command to build the rest of the stack

docker-compose -f docker-compose.yml build --no-cache --progress=plain

    docker-compose up -d --build



### Now create the database tables using the artisan command as shown below
**Migration Command**

    docker-compose exec <php service name> php artisan migrate

**Example**

    cd ./src
    docker-compose exec php php artisan migrate


### Now configure Laravel to work with Mongodb

**Laravel Configurations**

    cd ./src

**MongoDB Extension for Laravel**

    docker-compose run --rm composer require jenssegers/mongodb:^3.9 --ignore-platform-req=ext-mongodb

**Update .env**

    ....
    ....
    MONGO_CONNECTION=mongodb
    MONGO_HOST=mongo
    MONGO_PORT=27017
    MONGO_AUTH_DATABASE=admin
    MONGO_DATABASE=helloworld
    MONGO_USERNAME=root
    MONGO_PASSWORD='SdkW7BkXsKY6i82UuQ34'
    ....
    ....

Also, update the database configuration by updating config/database.php as shown below.

    ....
    ....
        'default' => env('DB_CONNECTION', 'mysql'),
        //'default' => env('DB_CONNECTION', 'mongodb'),
    ....
    ....
        'connections' => [
    ....
    ....
            'mongodb' => [
                'driver'   => 'mongodb',
                'host'     => env('MONGO_HOST'),
                'port'     => env('MONGO_PORT'),
                'database' => env('MONGO_DATABASE'),
                'username' => env('MONGO_USERNAME'),
                'password' => env('MONGO_PASSWORD'),
                'options'  => [
                    'database' => env('MONGO_AUTH_DATABASE')
                ]
            ],
    ....
    ....


Now, configure the providers to use the MongoDB extension provided by Jens Segers by updating the config/app.php file as shown below.

    ....
    ....
        'providers' => [
    ....
    ....
        Jenssegers\Mongodb\MongodbServiceProvider::class
        ],
    ....
    ....

Now, run the build and up commands to again build the application and launch it. We can access MongoDB using the URL - http://localhost:8081. It will ask for the basic authentication configured by us. The home page should be similar as shown below.


Stack Access:

The bare stack as built with these instructions, does not do anything but deliver a brand new Laravel 9 installation, (This too can be changed by editing the composer create-project above to any required version, although the MongoDB install, if still required, may take some coercion afterwards.

The elements, if installed as above, will be available as shown here:

Larvel (public folder): http://localhost/  
Mongo Express: http://localhost:8081/  
phpMyAdmin: http://localhost:8085/  
cAdvisor: http://localhost:8090/  
Prometheus: http://localhost:9090/  
Grafna: http://localhost:3000/  
  
Xdebug (remote client available via VSCode): http://localhost:9005

