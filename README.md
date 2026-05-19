### Building and running your application

When you're ready, start your application by running:
`docker compose up --build`.

Your application will be available at http://localhost:9001:80.


echo "Hello from Docker PHP!";
echo "first command: docker init";
echo "second command: docker compose up --build";
echo "check before the second command:docker version or docker info";
echo "for restart you can direct run docker down and then docker up --build";


ssl issue resolve=>

SSL is definitely being intercepted. Let's see exactly which certificate is being presented:=>
php -r "$ctx = stream_context_create(['ssl' => ['verify_peer' => false, 'capture_peer_cert' => true]]); file_get_contents('https://repo.packagist.org/packages.json', false, $ctx); $cert = stream_context_get_params($ctx); var_dump(openssl_x509_parse($cert['options']['ssl']['peer_certificate']));"

here its show that which one is intrupt the ssl traffic =>
in my case Confirmed — Zscaler is intercepting your SSL traffic. => then run this command=>
php -r "$ctx = stream_context_create(['ssl' => ['verify_peer' => false, 'capture_peer_cert_chain' => true]]); file_get_contents('https://repo.packagist.org/packages.json', false, $ctx); $params = stream_context_get_params($ctx); foreach($params['options']['ssl']['peer_certificate_chain'] as $i => $cert){ openssl_x509_export($cert, $out); file_put_contents('C:\\xampp\\php\\extras\\ssl\\zscaler_'.$i.'.pem', $out); echo 'Saved cert '.$i.'\n';}"

after this 3 Zscaler certs saved. Now append them all to our cacert.pem bundle:
type "C:\xampp\php\extras\ssl\zscaler_0.pem" >> "C:\xampp\php\extras\ssl\cacert.pem"
type "C:\xampp\php\extras\ssl\zscaler_1.pem" >> "C:\xampp\php\extras\ssl\cacert.pem"
type "C:\xampp\php\extras\ssl\zscaler_2.pem" >> "C:\xampp\php\extras\ssl\cacert.pem"

Now copy the updated bundle to the SSL directory too:
copy /Y "C:\xampp\php\extras\ssl\cacert.pem" "C:\Program Files\Common Files\SSL\cert.pem"

if Composer uses its own curl, not PHP's. Let's point Composer directly to our cert bundle:
composer config --global cafile "C:\xampp\php\extras\ssl\cacert.pem"


there is one more case if while installing composer require laravel cachier then if you get this error=
 curl error 60 while downloading https://repo.packagist.org/packages.json: SSL certificate problem: unable to get local issuer certificate

 then again run below commands=>
 docker compose cp "C:\xampp\php\extras\ssl\cacert.pem" server:/usr/local/share/ca-certificates/zscaler.crt
 docker compose exec server update-ca-certificates
 docker compose exec server bash -c "composer config --global cafile /etc/ssl/certs/ca-certificates.crt"
 docker compose exec server bash -c "composer require laravel/cashier"

done 
for cachier migration run this command with docker=>
docker compose exec server bash -c "php artisan vendor:publish --tag=cashier-migrations"

install laravel new project in this docker =>

docker compose down

then run =>
composer create-project laravel/laravel src "^12.0"
 
 remove the wrong failed folder, if it was created.Only run this if you see a laravel folder created by the failed command:->Remove-Item laravel -Recurse -Force


What you have so far ✅
my-docker-project/
├── src/              ← Your Laravel project lives here
├── Dockerfile        ← Instructions to build your PHP/Apache container
├── compose.yaml      ← Defines all your services (PHP, MySQL, Redis etc.)
├── .dockerignore     ← Files to ignore when building
└── README.Docker.md  ← Just a readme

What Docker is doing (simple explanation)
Think of it like this:

Dockerfile = recipe to build your app container
compose.yaml = list of all containers you need running together
Containers = mini computers running inside your PC

Your app needs 3 containers to work:

server → PHP + Apache (runs Laravel)
db → MySQL (stores data)
phpmyadmin → Visual database manager (like XAMPP's phpMyAdmin)
redis → Cache/queue driver

update .env file in the laravel root directory there is one new thing is added 

DB_HOST=db
 because now we set this db as docker creted configuration in compose.yaml file 

after this update the compose.yaml file and then run docker compose up --build

then also do one more thing that we have to run migration for this we exexute this cmd=>

"docker compose exec server php artisan migrate"


MySQL (db service) → uses a ready-made image:
yamlimage: mysql:8.0

We just download it directly from Docker Hub, no customization needed


PHP/Apache (server service) → uses a custom image:
yamlbuild:
  context: .

This means "build the image yourself using the Dockerfile in this folder"


So your Dockerfile IS the PHP server setup. It:

Downloads php:8.2.12-apache as base
Installs required PHP extensions (pdo, mysql, zip etc.)
Enables Apache mod_rewrite (needed for Laravel routes)
Points Apache to Laravel's public folder
Copies your src/ code into the container


Simple analogy:

MySQL image = buying a ready-made furniture from a shop
PHP/Apache image = building your own custom furniture using instructions (Dockerfile)


for phpmyadmin visual interface to manage your database 
add  in the compose.yaml  => phpmyadmin configuration and docker create image form phpmyadmin
