name: Laravel
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
jobs:
  laravel-tests:
    runs-on: ubuntu-latest
    services:
      postgresql:
        image: postgres:13
        env:
          POSTGRES_USER: root
          POSTGRES_PASSWORD: wWnF8Jm4HYD366vlzcUZEh4A7G6A8DOG
          POSTGRES_DB: ropahr
        ports:
          - 5432:5432
        options: >-
          --health-cmd="pg_isready"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=3

    steps:
    - uses: actions/checkout@v4
    - name: Setup PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.1'
        extensions: bcmath, ctype, fileinfo, json, mbstring, openssl, pdo, tokenizer, xml
    - name: Copy .env
      run: php -r "file_exists('.env') || copy('.env.example', '.env');"
    - name: Install Dependencies
      run: composer install --prefer-dist --no-interaction --no-scripts --no-progress
    - name: Generate Key
      run: php artisan key:generate
    - name: Set Permissions
      run: chmod -R 777 storage bootstrap/cache
    - name: Set up PostgreSQL
      run: |
        echo "DB_CONNECTION=pgsql" >> .env
        echo "DB_HOST=dpg-cpu2mt52ng1s73ea15rg-a.oregon-postgres.render.com" >> .env
        echo "DB_PORT=5432" >> .env
        echo "DB_DATABASE=ropahr" >> .env
        echo "DB_USERNAME=root" >> .env
        echo "DB_PASSWORD=wWnF8Jm4HYD366vlzcUZEh4A7G6A8DOG" >> .env
    - name: Create Database
      run: |
        psql -h dpg-cpu2mt52ng1s73ea15rg-a.oregon-postgres.render.com -U root ropahr -c 'CREATE DATABASE ropahr;'
        php artisan migrate --force
    - name: Run Tests
      run: php artisan test
