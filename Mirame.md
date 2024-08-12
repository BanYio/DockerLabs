# MIRAME
 ![image](https://github.com/user-attachments/assets/1ddcaf48-c3a5-42b7-a15e-f44f308afd90)

## Reconocimiento
 Empezaremos como siempre con un escaneo de puertos con la herramienta [autonmap](https://github.com/BanYio/AutoNMAP)
 ![image](https://github.com/user-attachments/assets/8a773d59-e068-484a-9c0a-036281d7e420)

**Puerto 80**

Cuando nos metemos al puerto 80, nos lleva a un panel de login.
![image](https://github.com/user-attachments/assets/8f035963-f420-4921-b9c8-1d053479c6e4)
 
Probamos credenciales típicas pero no hay suerte, por lo que vamos a intentar ver si es vulnerable a un SQL Inyection, y esto lo podemos ver insertando una comilla simple ‘, en este caso, lo he probado en username y password y nos aparece un error SQL;
 ![image](https://github.com/user-attachments/assets/65086820-d939-43ab-82cb-f5649309b86a)

Si probamos login bypass SQL como;
```shell
' or '1'='1
' or ''='
' or 1]%00
' or /* or '
```
Entrariamos sin problema, pero tras un rato enumerando no he conseguido nada, por lo que he pensado en comprometer la base de datos.
## EXPLOTACION
Para ello nos abrimos burpsuite y capturamos la `petición de login y la guardamos como request.txt
 ![image](https://github.com/user-attachments/assets/6aa5a87b-2219-4c7d-9e53-2927259c75d7)

Con la request guardada podemos utilizar sqlmap para comprometer la base de datos
```shell
sqlmap -r request.txt --dbs
```
![image](https://github.com/user-attachments/assets/29b8d720-eb81-4a36-a1ec-67861c70a963)

Podemos ver la base de datos users, vamos  a ver sus tablas
```shel
sqlmap -r request.txt -D users --tables
```
![image](https://github.com/user-attachments/assets/24b8cec9-8cb9-4e05-ac02-ebc008c7ee64)

Vemos la tabla usuarios, lo siguiente es ver sus columnas
 ```shell
 sqlmap -r request.txt -D users -T usuarios --columns
```
![image](https://github.com/user-attachments/assets/e22c87d7-16cc-4647-a014-e0f2c00b5676)

Y vemos 3 columnas; 
  - id
  - password
  - username

Por último dumpeamos la base de datos para ver que datos tiene
```shell
sqlmap -r request.txt -D users -T usuarios --dump
```
 ![image](https://github.com/user-attachments/assets/fe3b1de2-fb06-440f-b47e-56d4e35cf0b4)

Vemos usuarios y contraseñas, lo primero que hacemos es guardarnos esta información y probar logins tanto en la web como por ssh con hydra

![image](https://github.com/user-attachments/assets/7408786c-6838-433e-91db-415a68e44ac5)
 
Pero no hay suerte, parece que estas credenciales no nos sirven, pero si observamos, vemos que hay un un **user: directorio** y **password: directoriotravieso**

Vamos a comprobar por si acaso que no haya ningún directorio en la web con ese nombre
 ![image](https://github.com/user-attachments/assets/f88d3f59-5b25-4901-8a8d-e7336ce0eaef)

 ![image](https://github.com/user-attachments/assets/5f5628ed-a3cc-464e-aa15-d5a0bbaf4d73)

Vemos una imagen, nos la descargamos para ver si tiene información oculta, vamos a poner en practica nuestros conocimientos de estenografia.

Lo primero es revisar metadatos de la imagen con exiftool
![image](https://github.com/user-attachments/assets/b3ecc8a7-737d-4cc9-8d74-e37da53e7783)
 
No vemos nada interesante, vamos  a ver si  hay archivos ocultos en la imagen con steghide
 ![image](https://github.com/user-attachments/assets/8721d34d-8d14-49cf-8343-69efd62a615d)

Parece que tiene una passphrase, para poder crackearla, lo haremos con la herramienta stegseek
 ![image](https://github.com/user-attachments/assets/3918e485-3026-4cdd-b55e-74ed4efaf781)

Podemos ver que la passphrase es **chocolate**, y que dentro oculta un ocultito.zip, el cual dentro tiene un secret.txt

Vamos a extraer ese .zip con steeghide
 ![image](https://github.com/user-attachments/assets/de0a094a-8f3b-4236-93e7-d1f7ad1c24ad)

Cuando intentamos extraer el contenido del zip, podemos ver que nos pide nuevamente una pass, pero esta vez no es chocolate, por lo que vamos a crakear este zip con la herramienta de zip2john
 ![image](https://github.com/user-attachments/assets/c59eadf2-c617-4584-b110-b3869386c4fe)

Como podemos ver la pass de este .zip es **stupid1**

Vamos a ver lo que contiene el secret.txt
 ![image](https://github.com/user-attachments/assets/df9a7724-070c-4773-8877-55c1478d6760)

Dentro del secret.txt hay unas credenciales, por lo que vamos a probar si son válidas y efectivamente, tenemos credenciales para conectarnos por ssh
## ESCALADA DE PRIVILEGIOS
Una vez conectados al servidor, comenzamos a enumerar para la escalada de privilegios, y listando los binarios con permisos SUID, he encontrado que esta el binario find con estos permisos
 ![image](https://github.com/user-attachments/assets/473c65ae-3ce3-4a16-8e6f-3aae2b2be1b2)

Para escalar privilegios me ayudo de la herramienta searchbins, muy útil para este tipo de escaladas
 ![image](https://github.com/user-attachments/assets/cb27ebb0-332f-4835-bf56-34239128a855)

## Pwned!
Ya tenemos la maquina comprometida, en la cual hemos visto un poco de SQL Injection, estenografia, zip cracking y escalada de privilegios con binarios SUID, una maquina fácil, pero muy completa para aprender y asentar conocimientos.
