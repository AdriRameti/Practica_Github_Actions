# Práctica Github Actions
Realizado por: Adrián Ramos Ureña

## ¿Que és Github Actions?
GitHub Actions es una herramienta que permite reducir la cadena de acciones necesaria para la ejecución de código, mediante la creación de un de flujo de trabajo encargado del Pipeline. Siendo configurable para que GitHub reaccione a ciertos eventos de forma automática según nuestras preferencias.

Por lo tanto, GitHub Actions permite crear workflows que se puedan utilizar para compilar, testear y desplegar código. Además, da la posibilidad de crear flujos de integración y despliegue continuo dentro de nuestro repositorio.

Actions utiliza paquetes de códigos en los contenedores de Docker, los cuales se ejecutan en los servidores de GitHub y que, a su vez, son compatibles con cualquier lenguaje de programación. Esto hace que puedan funcionar con servidores locales y nubes públicas.

## Resultado de los Ultimos Test

<!---Start place for the badge -->


<!---End place for the badge -->

## Iniciamos el directorio
A partir del repositorio anexado, descargamos el código, el cual será la base a partir para poder realizar la práctica. Se nos quedará una estructura inicial como la siguiente:

Una vez tengamos el proyecto, inicializamos  un nuevo repositorio sobre la carpeta a nivel local.
git init
Inicializado el repositorio, realizaremos el primer push al repositorio

## Creamos el Workflow inicial
Primero que todo, crearemos el directorio .github/workflows donde ira situado el workflow.

En este Workflow, irán situados los jobs necesarios para realizar la práctica. Entre ellos se encuentra el job de linter que se encargará de verificar la sintaxis utilizada en los ficheros Javascript, el job de Cypress que se encargará de ejecutar los test de cypress que contiene el proyecto, el job de badge que se encargará de publicar en el readme del proyecto el badge que indicará si se han superado los test de cypress o no, el job de deploy que se encargará de publicar el proyecto en la plataforma vercel, un job de envío de notificaciones y un job customizado.

### Job de Linter
El job de Linter será el encargado de verificar, a partir de un archivo json donde se le especifica la verificación, toda la sintaxis de los archivos Javascript.
En este primer job, lo recorrerá en la última versión de ubuntu y tendrá 3 steps.El primer step, Checkout Code, se encargará de descargarse el código.El segundo, Dependencies, realizará la instalación de todas las dependencias necesarias.Por último, el tercero, Init Linter, se encargará de ejecutar la verificación de la sintaxis.

#### Comprobamos el job
Para ello realizaremos un push y veremos la ejecución y si funciona correctamente.



Como podemos observar, hay errores de sintaxis los cuales debemos solucionar.

#### Arreglamos los errores de sintaxis y volvemos a comprobar
Para ello debemos cambiar las comillas simples por comillas dobles, cambiar la declaración de variable var por let y por último estructurar bien el default del switch.

### Job Cypress 
El job de Cypress será el encargado de ejecutar los test de cypress que contiene el proyecto.
El job se recorrerá en la última versión de ubuntu y se requerirá del job anterior, es decir Linter_job. Este estará compuesto de 4 steps los cuales serán:

Checkout Code, que será el encargado de obtener el código.
Run Cypress Job que se encargará de ejecutar los test de Cypress. Este tendrá un id para poder imprimir en un archivo .txt los resultados haciendo referencia al id del step. También, aunque apareciese un error, continuará ya que le hemos especificado la opción de continue-on-error:true, ya que queremos que aunque se produzca un error este continúe. Le especificamos con el WITH, el archivo de donde partirá y que se realizará el build e iniciaremos con el start. Además este step tendrá una variable de entorno (ENV), donde declaramos el token de nuestro repositorio para la vinculación.
Save Test Results que será el encargado de guardar la salida del step anterior en un archivo result.txt
Upload Artifact este step actualizará la información del Artifact. Usaremos la acción de actions/upload-artifact@v2 , y declararemos el nombre y la dirección del archivo result.txt .


#### Verificamos el job
Como podemos observar nos da error en la entrada de los Endpoints así que habrá que localizarlo y corregirlo.


#### Corregimos los errores
Hemos localizado el error del Endpoint y es porque había un error en el nombre de POST ya que ponía POST0 en el archivo /pages/api/users/index.js

### Job Add Badge
El job de Add Badge  será el encargado de publicar en el readme del proyecto el badge que indicará si se han superado los tests de cypress o no.

Se ejecutará en la última versión de node y tendrá un condicional if:always() para que se ejecute siempre pase lo que pase. Además utilizaremos el needs ya que se requiere del resultado del job de Cypress

El job consta de 4 steps que son:
Checkout code: será el encargado de obtener el código
Download Changes of Artifact: Este descargará los cambios realizados en el job de cypress para tener los resultados y así poder mostrar un badge de success o uno de error
Create Output Based In the Artifact: En este Step crearemos un output que se utilizará en la acción personalizada y en el archivo javascript donde declaramos la funcionalidad para modificar el Readme.MD.
Create Badge of Result: Esté step partirá de la acción creada para modificar el readme y será el encargado de crear el badge a partir de la salid de información del job de cypress.
Upload The Changes In The Readme: En este step definiremos nuestro usuario y repositorio de trabajo de la práctica y realizaremos el push del nuevo Readme.MD 

#### Creamos el archivo .yml de nuestra acción
En este .yml, realizaremos una breve descripción y declararemos el input MY-CYPRESS-TEST, que será el output creado en el workflow.yml en el job de Add_Badge_Job y el step Create Output Based In The Artifact. 
Se recorrerá en node sobre el index.js que se ha creado al realizar el build con el comando :
ncc build index.js --license licenses.txt

#### Creamos el Index.js de la creación del Badge
Declararemos una variable denominada outcome, que estará asociada al output denominado anteriormente y dependiendo del resultado de este, se inyectara un badge de success o un badge de failure


#### Localización del Badge en el Readme
Situamos dos comentarios que serán los asociados en el index.js donde inyectamos el badge para que este se sitúe entre los dos y no modifique todo el Readme.

#### Verificamos que haya funcionado el job

### Vercel Job
Este job será el encargado de publicar el proyecto en la plataforma de vercel. Este se ejecutará tras el job de Cypress.

#### Cuenta en vercel
Crearemos una cuenta en Vercel y en el apartado de settings/tokens crearemos un token que lo vinculamos con nuestro repositorio de github.


#### Linkeamos la cuenta vercel al proyecto

####  Creamos los tokens en nuestro repositorio de trabajo de Github
Los 3 tokens nuevos creados son ORG_ID, PROJECT_ID, VERCEL_TOKEN y significan lo siguiente:
ORG_ID , PROJECT_ID: Estos tokens nos aparecen en un archivo de Vercel al realizar el link del proyecto con la cuenta.
VERCEL_TOKEN: Este token lo creamos en la plataforma Vercel para poder comunicar las dos plataformas

####  Creamos el Deploy Job
Como hemos dicho anteriormente, será el encargado de desplegar nuestra aplicación en la plataforma de Vercel. Este job se ejecutará en la última versión de ubuntu y requerirá del Job de Cypress para su funcionamiento. Este job consta de 2 steps que son:
CheckOut Code: Será el encargado de obtener el código
Deploy Our App On Vercel Platform: Utilizando una acción de Vercel y asociando todos los tokens mencionados anteriormente, será el que hará el despliegue de la aplicación

#### Comprobamos que haya ido bien
