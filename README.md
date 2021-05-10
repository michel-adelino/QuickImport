# QuickImport — Import transactions, invoices and bills into QuickBooks Desktop from Excel or CSV


QuickImport is a tool that helps you import invoices, bills, transactions,
customers and vendors into QuickBooks Desktop in multiple currencies in bulk.

### Features
- **Import of Journal Entries transactions, Single-line Invoices and bills**. Import creates customers, and vendors automatically.
- **Supported formats: csv, xlsx, xls**
- **Multicurrency support**. The base currency can be USD, EUR or anything else. The currency of all your accounts is detected automatically
  (after you import the Chart of accounts).
- **Cross Currency Transactions**. Transfer between accounts of different currencies goes through the Undeposited
  funds account. QuickImport doesn't affect the Undeposited funds balance because it uses the accurate historical exchange rate.
- **Multi-tenancy**: if you have multiple company files on the same computer, you can add them all to QuickImport.


## Getting started:
### Connect QuickImport to QuickBooks Desktop

**In QuickImport**:

Add a company file in Users, define username, password and specify the home currency.
It's recommended to specify the company file location if you are going to use multiple company files on the same computer.
Once it's done download the QWC file.

**In Quickbooks** click `File / Update Web Services`

Then Add an Application, in the file dialog select the downloaded QWC.
Then click Yes, then select "When company file is open" and click continue.
When it's done don't forget to specify the password that you defined in QuickImport.

<video src="import.mp4" controls width="100%" muted></video>

### How to import transactions from Excel into QuickBooks Desktop

## How to install QuickImport

### Docker setup

Here is docker-compose example with [traefik v1.7](https://doc.traefik.io/traefik/v1.7/)

1. Create `docker-compose.yml` with the following content:
```yaml
version: '3.3'

services:
    php:
        image: your-registry/QuickImport/app_php
        environment:
            APP_ENV: 'prod'
            DATABASE_URL: 'mysql://eqi:pass123@mysql:3306/QuickImport?serverVersion=mariadb-10.2.22'
            MAILER_DSN: 'smtp://localhost'
            # MAILER_DSN: 'ses://ACCESS_KEY:SECRET_KEY@default?region=eu-west-1'
            # MAILER_DSN: 'ses+smtp://ACCESS_KEY:SECRET_KEY@default?region=eu-west-1'
            DOMAIN: 'your-domain.com'
        depends_on:
            - mysql
        networks:
            - backend

    nginx:
        image: your-registry/QuickImport/app_nginx
        depends_on:
            - php
        networks:
            - backend
            - webproxy
        labels:
            - "traefik.backend=QuickImport-nginx"
            - "traefik.docker.network=webproxy"
            - "traefik.frontend.rule=Host:your-domain.com"
            - "traefik.enable=true"
            - "traefik.port=80"

    mysql:
        image: leafney/alpine-mariadb:10.2.22
        environment:
            MYSQL_ROOT_PWD: 'pass123'
            MYSQL_USER: 'eqi'
            MYSQL_USER_PWD: 'pass123'
            MYSQL_USER_DB: 'QuickImport'
        volumes:
            - ./db_data/:/var/lib/mysql
        networks:
            - backend

networks:
    backend:
    webproxy:
        external: true
```
2. Tweak the environment variables and run
```
docker-compose up -d
```
3. Check the logs and make sure migrations executed
```
docker-compose logs -f
```
4. Create a user
```
docker-compose exec php bin/console app:create-user user@example.com --password pass123
The user has been created with id 1
```
5. Done! Now open `https://your-domain.com` and log in with the user you just created.


### Development setup

1. Clone the repo
```
git clone https://github.com/your-username/QuickImport.git
```
2. Install packages with composer
```
composer install
```
3. Copy `.env` to `.env.local` and configure DATABASE_URL and DOMAIN
4. Recreate the database
```
bin/console doctrine:schema:drop --full-database --force \
&& bin/console doctrine:migrations:migrate -n
#fixtures
bin/console doctrine:fixtures:load
```
5. Create the user
```
bin/console app:create-user user@example.com --password pass123
```
6. Run the server
```
cd ./public
php -S 127.0.0.1:9090
```

### Tests
```
bin/phpunit
```

### phpstan
```
vendor/bin/phpstan analyse -c phpstan.neon
vendor/bin/phpstan analyse -c phpstan-tests.neon
```

### Lookup historical currency rate
``` 
bin/console app:currency:get USD --date 2020-03-05 --base HKD
```
