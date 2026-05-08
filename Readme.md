# Practica 4 Jetkins Alfonso Ballesteros 

## 0º Conceptos
### Que es Jenkins?
Jenkins es una herramienta de automatización usada para implementar procesos de integración y despliegue continuo (CI/CD).

Su función principal es ejecutar automáticamente tareas del desarrollo de software, como instalar dependencias, ejecutar tests, comprobar calidad del código o generar builds cada vez que un desarrollador sube cambios al repositorio. 

### Que tiene que ver Jenkins con GitHub?
Jenkins se integra muy bien con GitHub porque puede conectarse a un repositorio y detectar cambios en ramas o commits; cuando esto ocurre, lanza automáticamente el pipeline definido en el archivo Jenkinsfile. Gracias a esto se implementa una estrategia CI/CD (Continuous Integration / Continuous Delivery), donde el código se valida y construye de forma automática y continua, reduciendo errores y asegurando que la aplicación siempre pueda desplegarse correctamente.

## 1º Instalo Docker para instalar jetkins 
Instalo Docker para instalar jetkins dentro como un contenedor para ahorrar problemas de configuracion de la maquina

## 2º configuro el jetkinsfile

Tengo que crear un jetkinsfile con todo lo que se me pide en la carpeta backend.
Este fichero indica a jetkins que acciones deber realizar. 


```

pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
        timeout(time: 5, unit: 'MINUTES')
    }

    environment {
        FORCE_COLOR = '0'
        NO_COLOR = 'true'
    }

    stages {
        stage('Audit tools') {
            steps {
                dir('backend') {
                    sh 'node --version'
                }
            }
        }

        stage('Install dependencies') {
            steps {
                dir('backend') {
                    sh 'npm install'
                }
            }
        }

        stage('Format check') {
            steps {
                dir('backend') {
                    sh 'npm run format:check'
                }
            }
        }

        stage('Code quality') {
            steps {
                dir('backend') {
                    sh 'npm run lint'
                }
            }
        }

        stage('Type check') {
            steps {
                dir('backend') {
                    sh 'npm run type-check'
                }
            }
        }

        stage('Tests') {
            steps {
                dir('backend') {
                    sh 'npm run test'
                }
            }
        }

        stage('Build') {
            steps {
                dir('backend') {
                    sh 'npm run build'
                    archiveArtifacts artifacts: 'dist/**', fingerprint: true
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Review logs.'
        }

        always {
            cleanWs()
        }
    }
}
```

## Explicacion de cada punto del fichero jenkinsfile


---

## 1. `pipeline { ... }`

  ```
pipeline {
```

Define el inicio de una **pipeline declarativa**.

Una pipeline es una secuencia automatizada de tareas que Jenkins ejecuta.

---

## 2. `agent any`

  ```
agent any
```

Indica dónde se ejecutará la pipeline.

### Opciones comunes

| Opción           | Significado                                |
| ---------------- | ------------------------------------------ |
| `any`            | Puede ejecutarse en cualquier nodo Jenkins |
| `none`           | No asigna nodo automáticamente             |
| `label 'linux'`  | Solo en agentes con esa etiqueta           |
| `docker { ... }` | Ejecuta dentro de un contenedor Docker     |

En tu caso:

  ```
agent any
```

➡ Jenkins puede usar cualquier agente disponible.

---

## 3. `options`

  ```
options {
    disableConcurrentBuilds()
    timestamps()
    timeout(time: 5, unit: 'MINUTES')
}
```

Son configuraciones globales del pipeline.

---

### `disableConcurrentBuilds()`

  ```
disableConcurrentBuilds()
```

Evita que dos ejecuciones del mismo pipeline ocurran al mismo tiempo.

#### Ejemplo

Si alguien hace dos commits rápidos:

* Build #15 empieza
* Build #16 espera a que termine #15

Muy útil para evitar:

* conflictos
* despliegues simultáneos
* corrupción de artefactos

---

### `timestamps()`

  ```
timestamps()
```

Añade hora a cada línea del log.

#### Sin timestamps

```text
Running tests...
Build complete
```

#### Con timestamps

```text
[12:01:02] Running tests...
[12:01:40] Build complete
```

Muy útil para debugging.

---

### `timeout(...)`

  ```
timeout(time: 5, unit: 'MINUTES')
```

Cancela la pipeline si tarda más de 5 minutos.

Protege Jenkins de:

* builds colgados
* tests infinitos
* procesos bloqueados

---

## 4. `environment`

  ```
environment {
    FORCE_COLOR = '0'
    NO_COLOR = 'true'
}
```

Define variables de entorno globales.

---

### `FORCE_COLOR = '0'`

Desactiva colores ANSI en herramientas Node.js.

---

### `NO_COLOR = 'true'`

También deshabilita colores en la salida.

#### ¿Por qué?

Porque los logs Jenkins a veces muestran caracteres raros con colores ANSI.

---

## 5. `stages`

  ```
stages {
```

Aquí defines las fases del pipeline.

Cada `stage` representa un paso lógico.

---

## 6. Stage: Audit tools

  ```
stage('Audit tools') {
```

Comprueba herramientas instaladas.

---

### `dir('backend')`

  ```
dir('backend') {
```

Cambia temporalmente al directorio `backend`.

Equivalente a:

```bash
cd backend
```

---

### `sh 'node --version'`

  ```
sh 'node --version'
```

Ejecuta un comando shell Linux.

Aquí imprime la versión de Node.js.

---

## 7. Install dependencies

  ```
sh 'npm install'
```

Instala dependencias desde `package.json`.

Genera:

* `node_modules`
* dependencias necesarias para build/test

---

## 8. Format check

  ```
sh 'npm run format:check'
```

Normalmente ejecuta Prettier o similar.

Verifica formato del código.

Ejemplo:

* espacios
* indentación
* comillas
* saltos de línea

No modifica archivos; solo valida.

---

## 9. Code quality

  ```
sh 'npm run lint'
```

Ejecuta el linter (normalmente ESLint).

Busca:

* errores potenciales
* malas prácticas
* variables no usadas
* imports incorrectos

---

## 10. Type check

  ```
sh 'npm run type-check'
```

Verifica tipos TypeScript sin compilar.

Muy común en proyectos TS.

Detecta:

* tipos incompatibles
* funciones incorrectas
* objetos mal definidos

---

## 11. Tests

  ```
sh 'npm run test'
```

Ejecuta tests automáticos.

Puede usar:

* Jest
* Vitest
* Mocha
* etc.

Si falla un test:

➡ la pipeline falla.

---

## 12. Build

  ```
sh 'npm run build'
```

Compila la aplicación.

Ejemplo:

* TypeScript → JavaScript
* empaquetado frontend
* generación de `dist/`

---

## 13. `archiveArtifacts`

  ```
archiveArtifacts artifacts: 'dist/**', fingerprint: true
```

Guarda artefactos generados.

---

### `artifacts: 'dist/**'`

Archiva todo dentro de `dist`.

Ejemplo:

```text
dist/
  app.js
  bundle.js
```

---

### `fingerprint: true`

Genera hashes únicos de archivos.

Sirve para:

* trazabilidad
* detectar cambios
* relacionar builds

---

## 14. `post`

  ```
post {
```

Acciones que se ejecutan al finalizar la pipeline.

---

## 15. `success`

  ```
success {
    echo 'Pipeline completed successfully!'
}
```

Se ejecuta si TODO fue bien.

---

## 16. `failure`

  ```
failure {
    echo 'Pipeline failed. Review logs.'
}
```

Se ejecuta si algo falla.

---

## 17. `always`

  ```
always {
    cleanWs()
}
```

Se ejecuta siempre.

---

### `cleanWs()`

Limpia workspace Jenkins.

Borra:

* node_modules
* dist
* archivos temporales

Evita problemas entre builds.

---

## Flujo completo del pipeline

El orden real es:

```text
1. Elegir agente Jenkins
2. Aplicar opciones
3. Definir variables entorno
4. Audit tools
5. npm install
6. format check
7. lint
8. type-check
9. tests
10. build
11. guardar artifacts
12. success/failure
13. limpiar workspace
```

---

## Qué está validando esta pipeline

Tu pipeline está siguiendo una estrategia típica de CI moderna:

```text
Código correcto →
Formato correcto →
Lint correcto →
Tipos correctos →
Tests correctos →
Build correcto
```

Si cualquier paso falla:

➡ Jenkins detiene la pipeline.

---

## Posibles mejoras habituales

Algunas mejoras comunes serían:

### Cache de npm

  ```
npm ci
```

en lugar de:

  ```
npm install
```

Más rápido y reproducible.

---

### Publicar resultados de tests

  ```
junit 'reports/*.xml'
```

---

### Ejecutar stages en paralelo

Ejemplo:

* lint
* type-check
* tests

al mismo tiempo.

---

### Docker

Ejecutar en entorno reproducible:

  ```
agent {
    docker {
        image 'node:20'
    }
}
```













## 3. Ejecuto Jenkins en docker com este comando:

```

    docker run -d   --name jenkins   -p 8080:8080   -p 50000:50000   -v jenkins_home:/var/jenkins_home   jenkins/jenkins:lts

```


-p 8080:8080 y  -p 50000:50000  Ppra los puertos
-v jenkins_home:/var/jenkins_home   jenkins/jenkins:lts

para que mi directorio jetkins este unido con el suyo

![a](imgs/01-docker-jetkins.jpg)


Compruebo que esta corriendo bien 

```

    docker ps 

```

![a](imgs/02-docker-ps.jpg)


## 3. Nos vamos al localhost de jetkins 

![a](imgs/03-localhost-jetk.jpg)

Nos falta esa password y la buscamos con 

## 4. Obtener contraseña inicial

Ejecuto 

```bash 
        docker logs jenkins
        
```

![a](imgs/04-pass.jpg)



## 5. Instalamos jetkinsa
la copiamos en localhost para proceder a la instalación

![img](imgs/05-instalacionbasica.jpg)


## 6. Creamos usuario

![a](imgs/07-usuario.jpg)

Genial ya tenemos la configuracion en los volumenes 

-v jenkins_home:/var/jenkins_home   jenkins/jenkins:lts

pero esa no nos sirve ya que le falta node 



## 7 Creo una imagen con node para que pueda funcionar el proyecto


Pero la imagen de jetkins que tenemos no tiene node asi que vamos a crear una nueva con el asi que paramos la maquina y la borramos pero dejamos la imagen que vamos a usar de base y el volumen 

```
    docker stop jenkins

    docker rm jenkins

```

Como nuestro proyecto ya tiene un dockerfile montado pues vamos adelante 

```Dockerfile
FROM jenkins/jenkins:lts-jdk21

USER root

# Install docker-cli
RUN apt-get update && apt-get install -y lsb-release zip \
  && curl -fsSLo /etc/apt/keyrings/docker-archive-keyring.asc https://download.docker.com/linux/debian/gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker-archive-keyring.asc] https://download.docker.com/linux/debian $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list \
  && apt-get update && apt-get install -y docker-ce-cli

# Install docker-compose
RUN curl -fsL https://api.github.com/repos/docker/compose/releases/latest | grep tag_name | cut -d '"' -f 4 | tee /tmp/compose-version  \
  && mkdir -p /usr/lib/docker/cli-plugins  \
  && curl -fsSLo /usr/lib/docker/cli-plugins/docker-compose https://github.com/docker/compose/releases/download/$(cat /tmp/compose-version)/docker-compose-$(uname -s)-$(uname -m)  \
  && chmod +x /usr/lib/docker/cli-plugins/docker-compose  \
  && ln -s /usr/lib/docker/cli-plugins/docker-compose /usr/bin/docker-compose  \
  && rm /tmp/compose-version

# Install NodeJS
RUN apt-get update \
  && apt-get install -y ca-certificates curl gnupg \
  && curl -fsSL https://deb.nodesource.com/setup_lts.x | bash -  \
  && apt-get install -y nodejs

USER jenkins
```

Vamos a crear la imagen de jetkins + node y la llamamos jenkins-node

```
    docker build -t jenkins-node .
```
![img](imgs/08-dockerfile.jpg)

## 8 Ejecutamos jetkins con node dentro 

```
    docker run -d   --name jenkins   -p 8080:8080   -p 50000:50000   -v jenkins_home:/var/jenkins_home   jenkins-node
  
```
![a](imgs/09-dockerrun.jpg)

y nos logeamos en localhost

![a](imgs/10-login.jpg)


## 9 configurar Jetkins+github
Ya tenemos el jetkins instalado en local vamos a configurar el proyecto.

![a](imgs/11-jetkins-web.jpg)


## 10 Escogemos una multibranch
![a](imgs/12-multib.jpg)


## 11 En sources ponemos nuestro github

 ![a](imgs/13-configure-mbranch.jpg)

 ## 12 Guardamos y se pone a scanear nuestro github

![a](imgs/14-scan.jpg)

Aqui debe haber encontrado nuestra rama main 


 ## 13 Selecionando el Job
Nos vamos al panel central de jetkins y deberia salir algo asi 
Pulsamos en el job (A mi me sale Curso cep pq es el nombre que le puse antes)

![a](imgs/15-cp.jpg)

 ## 13 Haciendo el build 

ya pulsamos en build o construir 
![a](imgs/16-build.jpg)



 ## 14 Haciendo el build 

Podemos comprobar que todo ha ido bien en status que deberia salir algo asi y en el fichero server.mjs

![a](imgs/17-status.jpg)



 ## 15 Resumen 
El flujo funciona así cuando subos código a GitHub y Jenkins revisa automáticamente el repositorio buscando cambios. Cuando detecta una nueva versión del código (por ejemplo un commit en la rama main), ejecuta el Jenkinsfile, que contiene una serie de pasos automáticos llamados pipeline. En ese pipeline Jenkins comprueba que el proyecto funciona correctamente: instala dependencias, verifica el formato del código, analiza calidad, revisa tipos, ejecuta tests y finalmente genera la build de la aplicación. Si todo sale bien, guarda los artefactos generados (dist/server.mjs) y muestra un mensaje de éxito; si algo falla, detiene el proceso y muestra errores en los logs. 


## 15 Probamos que esta bien instalado
### Hago unos cambios 

![a](imgs/18-cambio.jpg)

### Hago  commit y push con los cambios 

```

git add .
git commit -m "Test Jenkins pipeline"
git push

```
![a](imgs/19-push-jk.jpg)

### Solicito un Scan Multibranch Pipeline Now

Voy a Jenkins → tu proyecto → rama main.
Pulsa: Scan Multibranch Pipeline Now

![a](imgs/20-sacn.jpg)

## 16 Comprobamos Console output

Pulso en  Console output de la ultima ejecucion 

![a](imgs/21-consoleoutput.jpg)

Compruebo que todo va bien

![a](imgs/22-ok.jpg)


