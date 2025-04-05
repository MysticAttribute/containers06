# Crearea unei aplicații multi-container

## Scopul lucrării: Famialiarizarea cu gestiunea aplicației multi-container creat cu docker-compose

### Sarcina propusa: De creat o aplicatie php pe baza a trei containere: nginx, php-fpm, mariadb, folosind docker-compose

### Mod de lucru

1. Cream un director *mounts/sites* in care includem fisierele unui site PHP creat de noi

2. Cream fisierul *.gitignore* si adaugam in el urmatorul continut:

    ```shell
    # Ignore files and directories
    mounts/site/*
    ```

3. Cream directorul *nginx*  iar in el un fisier *default.conf* cu urmatorul continut:

    ```shell
    server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        }
    }
    ```

4. Cream in directorul radacina *containers06* fisierul *docker-compose.yml* cu continutul dat:

    ```shell
    version: '3.9'

    services:
    frontend:
        image: nginx:1.19
        volumes:
        - ./mounts/site:/var/www/html
        - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
        ports:
        - "80:80"
        networks:
        - internal
    backend:
        image: php:7.4-fpm
        volumes:
        - ./mounts/site:/var/www/html
        networks:
        - internal
        env_file:
        - mysql.env
    database:
        image: mysql:8.0
        env_file:
        - mysql.env
        networks:
        - internal
        volumes:
        - db_data:/var/lib/mysql

    networks:
    internal: {}

    volumes:
    db_data: {}
    ```

5. Crearea fisierului *mysql.env* in radacina proiectului (*containers06*) si adaugam in el urmatoarele linii

    ```shell
    MYSQL_ROOT_PASSWORD=secret
    MYSQL_DATABASE=app
    MYSQL_USER=user
    MYSQL_PASSWORD=secret
    ```

6. Pornirea si testarea containerelor prin comanda:

    ```shell
    docker-compose up -d
    ```

### Intrebari

1. În ce ordine sunt pornite containerele?

    - Mai intai se porneste containerul cu baza de date (database-1), urmatorul se porneste containerul backend-1, iar la final se va porni frontend-1

2. Unde sunt stocate datele bazei de date?

    - Ele se stocheaza in volumul *db_data:/var/lib/mysql* ceea ce observam din liniile fisierului *docker-compose.yml* si anume *volumes: db_data:/var/lib/mysql* si *volumes: db_data: {}*

3. Cum se numesc containerele proiectului?

    - Containerele proiectului se numesc *backend-1*, *database-1* si *frontend-1*

4. Trebuie să adăugați încă un fișier app.env cu variabila de mediu APP_VERSION pentru serviciile backend și frontend. Cum se face acest lucru?

    - Mai intai cream un fisier *app.env* in care va fi o linie ca APP_VERSION=1.0 - de exemplu, iar apoi in fisierul *docker-compose.yml* in compartimentul env_file adaugam aceasta linie sub cea prezenta *- app.env*

## Concluzie

In urma efectuarii lucrarii de laborator am reusit sa cream o aplicatie multi-container, fiecare containerul cu rolul lui in baza aplicatiei integrale pe care o avem. Am creat un fisier *docker-compose.yml* care automat ne creeaza noua 3 containere, unul cu baza de date in baza imaginii *mysql:8.0*, unul backend pe baza imaginii *php:7.4-fpm* si unul frontend pe paza imaginii *nginx:1.19*, plus la aceasta am creat un volum unde se vor stoca fisierele bazei noastre de date care le putem gasi in Docker Desktop in compartimentul Volumes. Toate aceste 3 containere se afla intr-o retea a lor cu numele *internal* pentru ca sa fie accesibile unul pentru altul. Aceasta ne-a permis sa deschidem site-ul creat in baza orelor PHP pe localhost utilizand Docker ca mediu de testare neutru. In concluzie pot spune ca aceasta lucrare de laborator ne-a familiarizat cu o metoda simpla care automatizeaza procesul de creare si pornire a containerelor.
