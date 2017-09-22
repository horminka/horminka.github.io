---
layout: post
title: "Aplicaciones móviles con Ionic"
date: 2017-09-22 10:26:04 -0500
ref: ionic
lang: Español
categories: web
author: leonardo
image: http://inherit.mx/includes/frontend/inherit/media/press/58/creating-cross-platform-apps-using-ionic-framework.jpg
---

Ionic es un kit de desarrollo de software de código abierto que sirve para construir aplicaciones
móviles fácilmente. Una de las principales ventajas de Ionic es que
permite la creación de aplicaciones web progresivas y aplicaciones
móviles nativas para las principales tiendas de aplicaciones, utilizando
un mismo código.

![Ionic](http://inherit.mx/includes/frontend/inherit/media/press/58/creating-cross-platform-apps-using-ionic-framework.jpg)

Ionic está diseñado para crear aplicaciones que funcionan correctamente
y se ven bien en todos los dispositivos móviles y plataformas. Ionic
sigue los lineamientos de interfaz de usuario de cada plataforma y
utiliza SDKs nativos para  poder obtener las características y funcionalidades
de las aplicaciones nativas con la flexibilidad de las tecnologías web. Ionic
usa Cordova o Phonegap para correr de forma nativa, o se ejecuta en un
navegador como una aplicación web progresiva.

A continuación se presenta una guía para instalar Ionic en sistemas
Linux:

## Instalación

Para poder utilizar Ionic, es necesario instalar diferentes
dependencias, como NodeJS, JAVA, Android SDK y Cordova.

### NodeJS

Para instalar NodeJS, primero descargamos e instalamos Node Version Manager:

    $ wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.4/install.sh | bash

Luego instalamos y activamos la última versión de NodeJS:

    $ nvm install node
    $ nvm use node

### JAVA

Primero descargamos y descomprimimos la carpeta del [JDK Oracle](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).

Luego agregamos los ejecutables de este JDK como alternativas JAVA en el sistema operativo:

		$ sudo update-alternatives --install "/usr/bin/java" "java" "/home/citizen/jdk1.8.0_144/bin/java" 1
		$ sudo update-alternatives --install "/usr/bin/javac" "javac" "/home/citizen/jdk1.8.0_144/bin/javac" 1
		$ sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/home/citizen/jdk1.8.0_144/bin/javaws" 1
		$ sudo update-alternatives --install "/usr/bin/jarsigner" "jarsigner" "/home/citizen/jdk1.8.0_144/bin/jarsigner" 1

A continuación nos aseguramos que se estén utilizando las alternativas del JDK de Oracle que descargamos:

		$ sudo update-alternatives --config java
		$ sudo update-alternatives --config javac
		$ sudo update-alternatives --config javaws
		$ sudo update-alternatives --config jarsigner

Finalmente, se agrega la variable `JAVA_HOME` al entorno de trabajo:

		$ echo -e "export JAVA_HOME=$HOME/jdk1.8.0_144" >> .bashrc
		$ source ~/.bashrc

### Android SDK

Primero descargamos y descomprimimos la carpeta del SDK de Android:

		$ wget https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip
		$ mkdir android-sdk
		$ unzip sdk-tools-linux-3859397.zip -d android-sdk/

Luego creamos la variable de entorno `ANDROID_HOME` y la agregamos al `PATH`:

		$ echo -e "export ANDROID_HOME=$HOME/android-sdk" >> .bashrc
		$ echo -e "export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools" >> .bashrc
		$ source ~/.bashrc

Después ejecutamos el comando android para instalar los paquetes necesarios:

		$ ./android-sdk/tools/bin/sdkmanager "platform-tools" "platforms;android-25" "platforms;android-26" "build-tools;26.0.0"

### ADB y Gradle

A continuación instalamos estos dos paquetes necesarios para la interacción
con los dispositivos móviles y la compilación del código:

		$ sudo apt install android-tools-adb gradle

### Ionic Cordova

Para finalizar instalamos los paqueques Ionic y Cordova con `npm`:

    $ npm install -g ionic cordova

## Primera Aplicación

Para crear nuestra primera aplicación con Ionic utilizamos la interfaz
de línea de comandos:

    $ ionic start mi-app tabs

Esto nos crea el directorio `mi-app` que contiene los archivos de la aplicacion

Para generar una aplicación Android, lo único que debemos hacer es
entrar en la carpeta de la aplicación y por medio de la interfaz de
línea de comandos agregar esta plataforma a nuestro código:


		$ cd mi-app
		$ ionic cordova platform add android

A continuación podremos probar la aplicación en un dispositivo móvil
android:

		$ ionic cordova run android --device
