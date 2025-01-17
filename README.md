# Concrete CMS boilerplate theme - Work in progress

A fully featured Concrete CMS project comprising framework skeleton, custom theme, local Docker server and other
development tools.

Stack and technologies: WSL2, Concrete CMS, PHP8, MariaDB, Apache2, phpMyAdmin,
Composer, NPM, Sass, Gulp, PHPUnit, Prettier, Stylelint, ESLint

## Requirements

- WSL2 installed and enabled
- Docker Desktop for Windows installed
- Your project files should be located somewhere in WSL2 subsystem, for example in: `\\wsl$\Ubuntu\home\parasek\dev`
  which, under Linux, is accessible by `~/dev` path

## Installation

1. Open Windows Terminal, create and enter project folder in Linux home directory.

    ```
    cd ~/dev
    mkdir project_name
    cd project_name
    ```

2. Download files from GitHub.

    ```
    git clone https://github.com/parasek/concretecms-theme.git .
    ```

3. Remove `.git` folder.

    ```
    sudo rm -r .git
    ```

4. Copy `.env.dist` file to `.env`.

    ```
    cp .env.dist .env
    ```

5. <a name="first-installation-link"></a>If you are installing this Docker server for the first time, follow
   instructions (skip otherwise) in:

   > 🔗 [First Installation](#first-installation)

6. <a name="multiple-docker-servers-link"></a>If you want to run multiple Docker servers at the same time, follow
   instructions (skip otherwise) in:

   > 🔗 [Multiple Docker Servers](#multiple-docker-servers)

7. Copy `000-default.conf.example` file to `000-default.conf`.

    ```
    cp docker/web/apache2/sites-available/000-default.conf.example docker/web/apache2/sites-available/000-default.conf
    ```

8. Manually copy saved ssl certificates, that you generated earlier (skip this step if you did
   🔗 [First Installation](#first-installation) during this setup) to:

    ```
    docker/web/apache2/ssl/ssl_site.crt
    docker/web/apache2/ssl/ssl_site.csr
    docker/web/apache2/ssl/ssl_site.key
    ```

9. Set php version and timezone in `.env` file.

    ```
    APP_PHP_VERSION=8.3
    APP_TZ=Europe/Warsaw
    ```

10. Start Docker containers.

    ```
    docker compose up -d
    ```

11. Install Concrete CMS using Composer.

    Enter workspace container

    ```
    docker compose exec workspace bash
    ```

    Install Composer dependencies

    ```
    composer install -o
    ```

    Temporarily change name of live.database.php (installation won't start otherwise)

    ```
    mv public/application/config/live.database.php public/application/config/temp.database.php
    ```

    Install Concrete CMS \
    Remember to change fields below before you start installation: \
    --site: Site name \
    --language: Dashboard interface language \
    --site-locale: Main/first installed language on site \
    --timezone: Timezone, enter the same as APP_TZ in .env file \
    --admin-email: Admin account email \
    --admin-password: Admin account password \
    --starting-point: Set one from: theme / atomik_blank / atomik_full / elemental_full 

    ```
    php public/index.php c5:install --allow-as-root -n --db-server=mariadb --db-username=root --db-password=root --db-database=default --starting-point=theme --site="Sitename" --language=en_US --site-locale=en_GB --timezone=Europe/Warsaw --admin-email=example@email.com --admin-password="password"
    ```

    Revert name change of live.database.php (from now Concrete will be using live.database.php)

    ```
    mv public/application/config/temp.database.php public/application/config/live.database.php
    ```

    Remove original database.php file

    ```
    rm public/application/config/database.php
    ```

    Change required permissions (for localhost only)

    ```
    chmod -R 777 public/application/config public/application/files public/packages
    ```

12. Install NPM

    ```
    npm i
    ```

13. Generate css, js and other assets using Gulp tasks

    ```
    gulp build --prod
    ```

14. This probably good time to exit container, initialize git and make first commit if you are using GIT.

    ```
    exit
    ```

    ```
    git init
    ```
    
15. MailHog [http://localhost:8025](http://localhost:8025) is enabled at start. \
    It will catch emails send by your website and provide custom client. \
    Remember to disable it in .env file when your site goes live.
    ```
    # MAILHOG SETTINGS
    MAILHOG_ENABLED=0
    ```

16. Default links and login credentials:

    > Https url: [https://localhost:8100](https://localhost:8100) \
    PhpMyAdmin: [http://localhost:8200](http://localhost:8200) \
    Http url: [http://localhost:8300](http://localhost:8300) \
    MailHog server: [http://localhost:8025](http://localhost:8025)

    > Login credentials for phpMyAdmin/MySQL: \
    Server: mariadb \
    Username: root \
    Password: root \
    Database: default

## How to update Concrete CMS

1. Enter workspace container and run composer update:

    ```
    docker-compose exec workspace bash
    ```

    ```
    composer update concrete5/core
    ```

## How to change PHP version

1. Open .env and change php version (for example: 5.6, 7.4, 8.2 etc.).

    ```
    APP_PHP_VERSION=8.2
    ```

2. Rebuild web/workspace container

    ```
    docker compose build
    ```

    ```
    docker compose up -d
    ```

## Most used commands

1. In Linux Terminal:
   ```
   // You should be inside your project folder (where docker-compose.yml is)
   docker compose up -d // Start containers
   docker compose down // Stop and remove containers
   docker compose build // Rebuild containers (for example after changing php version)
   docker compose exec workspace bash // Enter workspace container (where you will be able to run build tasks etc.)
   
   // Anywhere on your computer
   docker exec -ti local-workspace bash // Alternative way to enter workspace container
   ```

2. Inside workspace container:

   ```
   // Custom commands that starts with "npm run" or "composer" are just "aliases".
   // You can find "real" commands inside "package.json" and "composer.json" files.
   
   exit // Exit container.
   
   // These are interchangeable ways to access Concrete binary.
   // Those will display a list of all available commands.
   php public/index.php
   ./vendor/bin/concrete5
   ./public/concrete/bin/concrete
   ./public/concrete/bin/concrete5
   php public/concrete/bin/concrete
   php public/concrete/bin/concrete5
   
   php public/index.php c5:config -g set concrete.maintenance_mode true // Enable maintenance mode.
   php public/index.php c5:config -g set concrete.maintenance_mode false // Disable maintenance mode.
   php public/index.php c5:ide-symbols // Generate helper files for IDE auto-completion.
   
   composer i -o // Install php packages listed in composer.json (with optimized flag).
   
   npm i // Install packages listed in package.json.
   npm update // Update packages listed in package.json.
   
   ######################
   # GULP tasks
   ######################
   
   // Source files are being stored in "./resources" folder.
   // Distribution file are being mostly stored in "./public/application/themes/theme/dist".
   
   gulp // Watch for changes in specified folders and perform related tasks.
   gulp watch // Same as above.
   gulp build // Conduct basic build tasks (scss, js, images, svg, favicons, translation).
   gulp build --prod // Same as above for live site (so with minification, without maps etc.).
   
   gulp scss // Build main css file.
   gulp js // Build main js file.
   gulp images // Compress images, minify svg files and copy them to "dist" folder.
   gulp svg // Build sprites from separate svg files, which then are loaded in "svg_sprites.php". 
   gulp favicons // Copy favicons to "dist" folder.
   gulp translation // Generate .mo files from .po files in ./public/application/languages/site.
   
   ########################
   # Js/CSS linters
   ########################
   
   You should probably configure your IDE, to lint your scss/js files on save.
   Though manual commands are always available.
   Those below are only "aliases", check "package.json" to see what they actually do.
   
   npm run eslint // Show potential js problems in "./resources/js" folder.
   npm run eslint:fix // Lint and show potential js problems in "./resources/js" folder.
   npm run stylelint // Show potential scss problems in "./resources/scss" folder.
   npm run stylelint:fix // Lint and show potential scss problems in "./resources/scss" folder.
   npm run prettier // Show list of file to lint using Prettier.
   npm run prettier:fix // Lint files in "./resources/js" and "./resources/scss" using Prettier.
   
   ########################
   # PHP-CS-Fixer
   ########################
   
   composer fix // Run PHP-CS-Fixer on all locations specified in .php-cs-fixer.php
   composer fix src // Run PHP-CS-Fixer on specific folder
   composer fix src/Foo/Bar/FooBar.php // Run PHP-CS-Fixer on specific file

   #########################
   # Testing
   #########################
   
   composer test // Run all tests
   composer test -- --filter testGetUserInfo // Run specific test

   #########################
   # Concrete settings
   #########################
   
   // Currently, I am using latest versions of PHP-CS-Fixer and PHPUnit
   // To be able to use fixer/run tests using Concrete configurations, 
   // you have to use those versions in composer.json:
   
   "phpunit/phpunit": "~4.3|^8.0",
   "mockery/mockery": "^0.9.9|^1.2",
   "friendsofphp/php-cs-fixer": "2.19.2" 
   
   // Delete composer.lock and run "composer i"
   // Copy phpunit.xml.dist from
   // https://github.com/concretecms/composer/tree/master
   
   php public/index.php c5:phpcs fix src // Run PHP-CS-Fixer using Concrete CMS settings
   composer test // Run tests using Concrete CMS version of PHPUnit
   ```

## <a name="first-installation"></a>First installation

1. Start Docker containers.

    ```
    cd ~/dev/project_name
    docker compose up -d
    ```

2. Enter workspace container.

    ```
    // From ~/dev/project_name folder (at the same level as docker-compose.yml)
    docker compose exec workspace bash

    // From any folder
    docker exec -ti local-workspace bash
    ```

3. Inside web container create ssl certificates for localhost domain
   (in Terminal run every command one by one).

    ```
    openssl genrsa -out "/etc/apache2/ssl/ssl_site.key" 2048
    openssl rand -out /root/.rnd -hex 256
    openssl req -new -key "/etc/apache2/ssl/ssl_site.key" -out "/etc/apache2/ssl/ssl_site.csr" -subj "/CN=localhost/O=LocalServer/C=PL"
    openssl x509 -req -days 7300 -extfile <(printf "subjectAltName=DNS:localhost,DNS:*.localhost") -in "/etc/apache2/ssl/ssl_site.csr" -signkey "/etc/apache2/ssl/ssl_site.key" -out "/etc/apache2/ssl/ssl_site.crt"
    chmod 644 /etc/apache2/ssl/ssl_site.key
    exit
    docker compose down
    ```

4. Add generated `ssl_site.crt` to Trusted Certificates on Windows 10:

    - Press Windows button and run `cmd`
    - In cmd.exe window type `mmc` and press enter to open Microsoft Management Console, allow it to make changes
    - Select `File -> Add/Remove Snap-in`
    - Select `Certificates` in a left window and click `Add`
    - Click `Finish`
    - Click `OK`
    - Expand tree on the left and go
      to `Console Root/Certificates - Current User/Trusted Root Certification Authorities/Certificates`
    - Right click `Certificates` and select `All Tasks -> Import...`
    - `Next`
    - Select generated `ssl_site.crt` (from `\\wsl$\Ubuntu\home\parasek\dev\project_name\docker\web\apache2\ssl` path)
    - `Next`, `Next`, `Finish`, `Yes`
    - You can close window without saving<br/><br/>

    ```
    IMPORTANT!
    Copy/save generated files somewhere on your computer.
    You will be using them everytime you create new project.
    - docker/web/apache2/ssl/ssl_site.crt
    - docker/web/apache2/ssl/ssl_site.csr
    - docker/web/apache2/ssl/ssl_site.key
    ```

⬅ [Go back to Installation](#first-installation-link)

## <a name="multiple-docker-servers"></a>Multiple Docker Servers

1. If you want to run multiple Docker servers at the same time, you have to set unique name/ports in .env file, for
   example:

    ```
    APP_NAME=othername
    APP_PORT_SSL=8101
    APP_PMA_PORT=8201
    APP_PORT=8301
    APP_DB_PORT=3307
    MAILHOG_HTTP_PORT=8026
    MAILHOG_SMTP_PORT=1026
    ```

   Your site will be accessible through:

   > Https url: [https://localhost:8101](https://localhost:8101) \
   PhpMyAdmin: [http://localhost:8201](http://localhost:8201) \
   Http url: [http://localhost:8301](http://localhost:8301) \
   MailHog server: [http://localhost:8026](http://localhost:8026)

   ⬅ [Go back to Installation](#multiple-docker-servers-link)
