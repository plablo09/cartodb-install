cartodb-install
===============

###Instalación de cartodb en ubuntu 12.04
####Clonamos el repositorio (git) de cartodb:

````
sudo apt-get install git
git clone --recursive https://github.com/CartoDB/cartodb.git

````
####Agregamos los repositorios de cartodb:

Antes, instalamos python-software properties para facilitarnos la vida:
````
sudo apt-get install python-software-properties
````
Luego agregamos los repositorios (uno por uno)
```
sudo add-apt-repository ppa:cartodb/base
sudo add-apt-repository ppa:cartodb/mapnik
sudo add-apt-repository ppa:cartodb/nodejs
sudo add-apt-repository ppa:cartodb/redis
sudo add-apt-repository  ppa:cartodb/postgresql-9.3
sudo add-apt-repository  ppa:cartodb/varnish
```
Actualizamos:
````
sudo apt-get update
````
#### Instalamos y configuramos las dependencias:

````
sudo apt-get install unp zip
````
Dependencias de cartodb:

````
sudo apt-get install libgeos-c1 libgeos-dev
sudo apt-get install gdal-bin libgdal1-dev
sudo apt-get install libjson0 python-simplejson libjson0-dev
sudo apt-get install proj-bin proj-data libproj-dev
````
Ahora Postgres:


````
sudo apt-get install postgresql-9.3 postgresql-client-9.3 postgresql-contrib-9.3 postgresql-server-dev-9.3
sudo apt-get install postgresql-plpython-9.3
````
Hay que editar el `pg_hba.conf` y cambiar todas las conecciones a trust:
````
sudo nano /etc/postgresql/9.3/main/pg_hba.conf
````
Cambia todo lo que diga peer o md5 a trust y reinicia el servicio:

````
sudo service postgresql restart
````
Ahora vamos a instalar postgis y configurar el template correspondiente:

````
sudo apt-get install postgresql-9.3-postgis-2.1
````
Para configurar facilmente el `template_postgis` vamos a crear un script:

````
nano template_postgis.sh
````
Copia el siguiente código en el archivo que acabas de crear:

````
#!/usr/bin/env bash
POSTGIS_SQL_PATH=`pg_config --sharedir`/contrib/postgis-2.1
createdb -E UTF8 template_postgis
createlang -d template_postgis plpgsql
psql -d postgres -c \
 "UPDATE pg_database SET datistemplate='true' WHERE datname='template_postgis'"
psql -d template_postgis -c "CREATE EXTENSION postgis"
psql -d template_postgis -c "CREATE EXTENSION postgis_topology"
psql -d template_postgis -c "GRANT ALL ON geometry_columns TO PUBLIC;"
psql -d template_postgis -c "GRANT ALL ON spatial_ref_sys TO PUBLIC;"
````
Damos permiso de ejecución al archivo:
````
chmod +x  template_postgis.sh
````
Ejecutamos el script como el usuario postgres:

````
sudo su - postgres
./template_postgis.sh
exit
````
Ahora vasmos a instalar la extensión schema triggers.
Primero instalamos mercurial para poder clonar el repositorio y make para
poder compilar:

```
sudo apt-get install mercurial
sudo apt-get install make
```
Clonamos el repositorio e instalamos:


```
hg clone https://bitbucket.org/malloclabs/pg_schema_triggers
cd pg_schema_triggers
make
sudo make install
```
Ahora agregamos `schema_triggers.so` a la sección preshared libraries
de postgresql.conf y reiniciamos el servicio:

```
echo "shared_preload_libraries = 'schema_triggers.so'" |
    sudo tee -a /etc/postgresql/9.3/main/postgresql.conf &&
    sudo service postgresql restart

cd ..
```

Añadimos el esquema cartodb al path de búsqueda de postgres. Edita el archivo postgresql.conf
y agrega cartodb a la sección search_path (posiblemente tengas que descomentarla).
Reinicia postgres.

####Instalamos Ruby:

````
curl -L https://get.rvm.io | bash
source .rvm/scripts/rvm
rvm install 1.9.3
````

####Instalamos Node.js:
````
sudo apt-get install nodejs npm
````
Para estar seguros de que estamos usando la versión corre4cta de node.js
instalamos NVM:

````
curl https://raw.githubusercontent.com/creationix/nvm/v0.12.0/install.sh | bash
source .bash_profile
nvm install v0.8.9
nvm use 0.8.9
````
####Instalamos Redis:
````
sudo apt-get install redis-server
````

####Ahora las dependencias de python:
Algunas dependencias del sistema:
````
sudo apt-get install python2.7-dev
sudo apt-get install python-gdal
sudo apt-get install build-essential
sudo apt-get install python-setuptools

````
Instalamos pip:

````
sudo easy_install pip
````
Instalamos las dependencias de python, como ya tenemos instalado python-gdal,
necesitamos comentar la linea `gdal==1.10.0` en cartodb/python_requirements.txt:

````
cd cartodb
sudo pip install --no-use-wheel -r python_requirements.txt

````
####Instalamos Varnish:
````
sudo apt-get install varnish
````
Es necesario configurar Varnish para que no use ssl. Editamos el archivo
`/etc/dafault/varnish` y elminamos la linea `-S /etc/varnish/secret`
de la sección DAEMON_OPTS (la que no está comentada). Reiniciamos varnish:

````
sudo service varnish restart
````
####Instalamos Mapnik:

````
sudo apt-get install libmapnik-dev python-mapnik mapnik-utils
````

####Instalamos CartoDB SQL API:
Asegurate de estar en el directorio home, `cd ~`
````
git clone git://github.com/CartoDB/CartoDB-SQL-API.git
cd CartoDB-SQL-API
git checkout master
npm install
````
Nos aseguramos de que exista el archivo development.js:

````
cp config/environments/development.js.example config/environments/development.js
````
Configuramos el api para que escuche en cualquier host:

````
perl -p -i -e 's/^(module.exports.node_host).*$/\1 = '"''/g" config/environments/development.js
````

####Instalamos Windshaft-cartodb
Desde el directorio home:

````
git clone git://github.com/CartoDB/CartoDB-SQL-API.git
cd Windshaft-cartodb
git checkout master
npm install
````
Creamos el archivo develpment.js:

````
cp Windshaft-cartodb/config/environments/development.js.example Windshaft-cartodb/config/environments/development.js
````
Y lo editamos para que escuche en cualquier host:
````
perl -p -i -e 's/^(    ,host:).*$/\1 '"''/g" config/environments/development.js
````
Ahora necesitamos hacerle creer a Windshaft que la versión de mapnik es 2.1.1 (hack medio raro):

````
perl -p -i -e 's/^(    ,mapnik_version:).*$/\1 '"'2.1.1'/g" config/environments/development.js

````
Creamos el directorio logs (adentro de Windshaft):

````
mkdir logs
````

####Instalamos ImageMagick:
````
sudo apt-get install imagemagick
````

####Instalación y configuración de cartodb

En una nueva terminal iniciamos redis:
````
redis-server
````

De regreso en la terminal original, en la carpeta cartodb (`cd ~/cartodb`),
exportamos el dominio que vamos a usar (este será también el usuario configurado)
e instalamos cartodb:

````
export SUBDOMAIN=development
rvm use 1.9.3@cartodb --create && bundle install
mv config/app_config.yml.sample config/app_config.yml
mv config/database.yml.sample config/database.yml

````
Si vas a usar como domnio para la aplicación localhost.lan, entonces no es necesario
editar la configuración.

Ahora necesitamos agregar una entrada a /etc/hosts para apuntar al dominio de la aplicación:
````
echo "127.0.0.1 ${SUBDOMAIN}.localhost.lan" | sudo tee -a /etc/hosts
````
Instalamos la extensión pg-cartodb desde la carpeta de cartodb:


````
cd lib/sql/
make all
sudo make install
````

Ahora ejecutamos el script para crear el usuario development (recuerda tener redis corriendo
  en otra terminal):

````
sh script/create_dev_user ${SUBDOMAIN}
````

Detenemos redis (ctrl-C en la terminal donde está corriendo)

Ejecutamos carto db:

````
bundle exec foreman start -p 3000
````

Ahora, en otra terminal:

````
rvm use 1.9.3@cartodb --create && bundle install
sh cartodb/script/restore_redis
````

Listo! Ahora te puedes ingresar a cartodb en development.localhost.lan:3000/login
usando las credenciales que configuraste
