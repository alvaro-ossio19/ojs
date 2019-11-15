# Open Journal Systems 3.1.2 (SIDISI)

> Open Journal Systems (OJS) has been developed by the Public Knowledge Project. For general information about OJS and other open research systems, visit the [PKP web site][pkp].

[![Build Status](https://travis-ci.org/pkp/ojs.svg?branch=master)](https://travis-ci.org/pkp/ojs)

## Documentation

You will find detailed guides in [docs](docs) folder.

## Using Git development source

Checkout submodules and copy default configuration :

    git submodule update --init --recursive
    cp config.TEMPLATE.inc.php config.inc.php

Install or update dependencies via Composer (https://getcomposer.org/):

    composer --working-dir=lib/pkp update
    composer --working-dir=plugins/paymethod/paypal update
    composer --working-dir=plugins/generic/citationStyleLanguage update

Install or update dependencies via [NPM](https://www.npmjs.com/):

    # install [nodejs](https://nodejs.org/en/) if you don't already have it
    npm install
    npm run build

If your PHP version supports built-in development server :

    php -S localhost:8000

See [Wiki][wiki-dev] for more complete development guide.

## PKP dev

Documentación de desarrollo:

    https://docs.pkp.sfu.ca/dev/documentation/en/getting-started

## Git Repository Fork: Vincular proyecto con repositorio padre

Verificar los repositorios remotos del proyecto:

    $ git remote -v
    > origin  https://github.com/alvaro-ossio19/ojs.git (fetch)
    > origin  https://github.com/alvaro-ossio19/ojs.git (push)

Si no existe el repositorio original, hacer lo siguiente:

    $ git remote add upstream https://github.com/pkp/ojs.git

Volvemos a verificar los repositorios remotos:

    $ git remote -v
    > origin        https://github.com/alvaro-ossio19/ojs.git (fetch)
    > origin        https://github.com/alvaro-ossio19/ojs.git (push)
    > upstream      https://github.com/pkp/ojs.git (fetch)
    > upstream      https://github.com/pkp/ojs.git (push)

Hacemos lo mismo para las librerías pkp y ui-library:

    $ cd lib/pkp
    $ git remote add upstream git@github.com:pkp/pkp-lib.git
    $ cd ../ui-library
    $ git remote add upstream git@github.com:pkp/ui-library.git

## Git Repository Fork: Sincronizar actualizaciones del repositorio padre

Vamos a la rama stable-3_1_2 (bifurcación):

    $ git remote update
    $ git fetch
    $ git checkout --track origin/stable-3_1_2

Para fusionar los cambios del repositorio padre desde upstream/stable-3_1_2 con la rama origin/stable-3_1_2 (bifurcación):

    $ git pull upstream stable-3_1_2
    $ git push

Para actualizar la librería pkp:

    $ cd lib/pkp
    $ git remote update
    $ git fetch
    $ git checkout stable-3_1_2
    $ git pull upstream stable-3_1_2
    $ git push

Para actualizar la librería ui-library:

    $ cd ../ui-library
    $ git remote update
    $ git fetch
    $ git checkout stable-3_1_2
    $ git pull upstream stable-3_1_2
    $ git push

Después de actualizar las librerías, los sincronizamos con OJS a su versión adecuada:

    $ git submodule update --init --recursive

Actualizamos las dependencias y compilamos la aplicación JavaScript:

    $ composer --working-dir=lib/pkp update
    $ npm install
    $ npm run build

Si se debe actualizar la base de datos, en el archivo config.ing.php cambiar installed = On a Off y ejecutar:

    $ php tools/upgrade.php upgrade

Luego volver a cambiar installed = Off a On.

## Upgrade

Para actualizar la aplicacion OJS desde una base de datos existente:

    https://pkp.sfu.ca/ojs/UPGRADE

## OJS: Revisión Académica

Esta versión OJS está personalizada para SIDISI. Se usará para las revisiones académicas de proyectos por parte de las Unidades de Gestión.

Después de realizar la clonación del repositorio, se debe dar permisos de escritura a las carpetas cache/ y public/

    chmod -R 777 cache/
    chmod -R 777 public/

Se debe evitar el borrado de los archivos .gitignore:

    chmod -w cache/t_cache/.gitignore
    chmod -w cache/t_compile/.gitignore
    chmod -w cache/t_config/.gitignore
    chmod -w cache/_db/.gitignore
    chmod -w cache/.gitignore
    chmod -w public/.gitignore

Creamos el archivo de configuración y damos permiso de escritura:

    cp config.TEMPLATE.inc.php config.inc.php
    chmod 777 config.inc.php

Ahora accedemos al ojs desde el navegador y realizamos la instalación.

Ejecutamos los siguientes queries en la db para el plugin LDAP UPCH:

    UPDATE users SET auth_id=NULL WHERE user_id=1;
    INSERT INTO auth_sources(auth_id,title,plugin,auth_default) VALUES(1,'LDAP UPCH','ldap-upch',1);

Por último, las nuevas columnas para SIDISI:

    alter table authors add column prot_participant_id bigint(20) null;
    alter table authors add column repository_role_id tinyint null;
    alter table authors add column sidisi_role_label varchar(128) null;

Con eso nos aseguramos que el usuario admin no iniciará sesión con LDAP.

Finalmente, si el servidor ya cuenta con un certificado digital SSL, entrar al archivo config.inc.php y cambiar las siguientes ; Force SSL connections site-wide 

    force_ssl = On

    ; Force SSL connections for login only
    force_login_ssl = On

## Agregar submódulos GIT

Para instalar nuevos repositorios como parte de OJS, se deben agregar como submódulos.
Tomamos como ejemplo el repositorio del plugin LDAP UPCH y lo agregamos de la siguiente manera en la raíz de OJS:

    $ git submodule add https://github.com/alvaro-ossio19/ojs-ldap.git  plugins/auth/ldap-upch

Esto habrá agregado al plugin como submodulo en OJS en el archivo .gitmodules:

    [submodule "plugins/auth/ldap-upch"]
	path = plugins/auth/ldap-upch
	url = https://github.com/alvaro-ossio19/ojs-ldap.git

Luego ejecutar en terminal:

    $ git submodule update --init --recursive

## Cambiar rama de un submódulo GIT

Si queremos utilizar una rama específica de un submódulo (en este ejemplo es v1.0), debemos hacer los siguiente:

    $ cd submodule_directory
    $ git checkout v1.0

Luego volvemos a la raíz de OJS y ejecutamos:

    $ git add submodule_directory
    $ git commit -m "moved submodule to v1.0 branch"
    $ git push


## Running Tests

We recommend using [Travis](https://travis-ci.org/) for continuous-integration
based testing. Review the Travis configuration file (`.travis.yml`) as a
reference for running the test locally, should you choose to do so.

The tests include an integration test suite that builds a data environment from
scratch, including the installation process. (This is the `-b` flag to the test
script `lib/pkp/tools/runAllTests.sh`; this is also executed in the Travis
environment.)

## Bugs / Issues

See https://github.com/pkp/pkp-lib/#issues for information on reporting issues.

## License

This software is released under the the [GNU General Public License][gpl-licence].

See the file [COPYING][gpl-licence] included with this distribution for the terms
of this license.

Third parties are welcome to modify and redistribute OJS in entirety or parts
according to the terms of this license. PKP also welcomes patches for
improvements or bug fixes to the software.

[pkp]: http://pkp.sfu.ca/
[readme]: docs/README.md
[wiki-dev]: http://pkp.sfu.ca/wiki/index.php/HOW-TO_check_out_PKP_applications_from_git
[php-unit]: http://phpunit.de/
[gpl-licence]: docs/COPYING
