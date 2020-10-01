## Step by Step
Langkah - langkah

1. buat/clone project laravel -> versi 5.8
2. konfigurasi file .travis.yml
```
# os yang di gunakan
os:
  - linux
# bahasa
language: php
# distro 
dist: trusty

php:
  - '7.1'
# service yang digunakan
services:
  - docker
# grouping  task
jobs:
  include:
  # melakukan testing 
    - stage: "Tests"                
      name: "Unit Test PHP"  
      script: 
      - travis_retry composer self-update
      - travis_retry composer install --prefer-source --no-interaction
      - cp .env.example .env
      - php artisan key:generate
      - vendor/bin/phpunit tests/Feature/ExampleTest.php
    # melakukan build images dan publish ke docker hub
    - stage: "Build Docker Image"
      name: "Build Images Docker" 
      script:
      - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
      - docker build -t travis-ci-build-stages-demo .
      - docker images
      - docker tag travis-ci-build-stages-demo $DOCKER_USERNAME/training_travis
      - docker push $DOCKER_USERNAME/training_travis
```


3. konfigurasi file Dockerfile
```
# using this awesome prebuild image:
FROM '123majumundur/php-7.1-nginx:cicd'
MAINTAINER Robby Dwi <robbie.developer@gmail.com>

# Install prestissimo for faster deps instalation 
RUN composer global require hirak/prestissimo

# Make directory for hosting the apps
RUN mkdir /home/app/app
WORKDIR /home/app/app

# Install dependencies
COPY composer.json composer.json
RUN composer install --prefer-dist --no-scripts --no-dev --no-autoloader && rm -rf /home/app/.composer

# Copy codebase
COPY --chown=app:root . ./

# Finish composer
#RUN composer dump-autoload
RUN composer dump-autoload --no-scripts --no-dev --optimize

EXPOSE 8080
```
4. setting environtment variable di travis-ci untuk memberikan nilai di .travis.yml
$DOCKER_USERNAME
$DOCKER_PASSWORD

5. push ke github
6. tunggu hingga proses testing dan build berhasil
7. cek di docker hub apakah repository punya kita sudah ter-push
8. pull repository kita -> docker pull ilhamfadhilah/training_travis
9. pastikan key di laravel sudah ter generate. ada di file .env nama key nya APP_KEY=
10. jalankan docker -> docker run -p 0.0.0.0:8080:8080/tcp --env-file ./.env ilhamfadhilah/training_travis