Psycho
![image](https://github.com/user-attachments/assets/fb7f7133-9e08-47fd-8870-8d8d45f8bf9a)

Reconocimiento:
Para comenzar, lo primero que haremos es realizar un escaneo de puertos con nmap, para ver que puertos y servicios tiene abiertos esta máquina.
 ![image](https://github.com/user-attachments/assets/df8d4135-1af0-4666-aef6-18b3c3893e13)

Puerto 80
Lo primero enumeramos las versiones de este servidor web para buscar posibles exploits, pero no es el caso
 ![image](https://github.com/user-attachments/assets/dcbf2581-9b57-4eb3-b8ae-788517d970f7)

Lo siguiente es realizar un poco de Fuzzing Web para la enumeración de directorios del servidor web.
![image](https://github.com/user-attachments/assets/82ac3b6d-a434-401e-a184-ec3438b46c85)

No encontramos gran cosa, pero me llama la atención el index.php, y en este .php aparece un error, lo que nos da a pensar que necesita un parámetro válido.
 ![image](https://github.com/user-attachments/assets/adabcdde-94aa-4ee0-8611-74fc3d0794d9)

Realizamos un fuzzing a un posible parámetro para ver si es vulnerable a un LFI.
 ![image](https://github.com/user-attachments/assets/56913acb-3105-442c-9cee-e8e8835252ae)

Explotación
Comprobamos que efectivamente, es vulnerable a un LFI si utilizamos el parámetro secret.
![image](https://github.com/user-attachments/assets/c082e957-80ea-4127-a76a-f5547c44e437)

Podemos observar que hay 2 usuarios, luisillo y vaxei. 
Recordando la enumeracion de puertos, tenemos el puerto 22 abierto, con el servicio ssh corriendo, por lo que vamos a ver si tenemos acceso a la clave id_rsa de alguno de estos 2 usuarios.
![image](https://github.com/user-attachments/assets/c56c67c0-8d6f-4374-a1f0-a616a35c1347)

Tenemos la clave id_rsa del usuario vaxei, nos la guardamos y le damos los permisos necesarios e intentamos conectarnos por ssh.
 ![image](https://github.com/user-attachments/assets/e551ca96-07e2-4618-9663-a4dd8a8b731f)

Escalada de privilegios
Ya tenemos acceso al servidor, ahora tenemos que escalar privilegios para comprometer por completo esta máquina.
Podemos observar que podemos ejecutar el binario perl con privilegios del usuario luisillo.
 ![image](https://github.com/user-attachments/assets/bb3be5d4-4602-45af-b6d5-0b0cf2524672)
 
Ya estamos como el usuario luisillo, y nuevamente con el comando sudo -l, vemos que puede ejecutar un script de python con privilegios de root.
 ![image](https://github.com/user-attachments/assets/27a8e41f-cfa3-4473-bf96-c97810b74327)
 ![image](https://github.com/user-attachments/assets/0a049f6e-1cf8-4d11-a28a-1f7c87571a1f)
 ![image](https://github.com/user-attachments/assets/3c20d3ba-c55b-4fe3-8570-d04aef111963)

Podemos ver que el script utiliza varias librerías, pero al no tener permiso de escrituras sobre este script ni en las propias librerías almacenadas en /usr/lib/python3.12, no lo podemos modificar, por lo que vamos a intentar a ver si es posible realizan un Python Library Hijacking, para ello he utilizado una el siguiente artículo, centrándonos en el segundo método que enseña; 
https://www.hackingarticles.in/linux-privilege-escalation-python-library-hijacking/
Lo primero es realizar un .py en la misma carpeta en la que se encuentra el script, /opt/, con el nombre de alguna de las librerías utilizadas en el script, en mi caso lo he hecho con subprocess.py
 ![image](https://github.com/user-attachments/assets/512891bd-7193-4be3-aef0-56affca75418)

 
En el fichero que he creado, he importado la librería os y ejecuto el comando id. Ejecuto con sudo el script y vemos como nos muestra el comando que hemos puesto en nuestra "librería falsa".
Para escalar privilegios se puede hacer de varias formas, en mi caso le voy a dar permisos SUID a /bin/bash, y para ello, modificamos nuestro subprocess.py y le añadimos "chmod u+s /bin/bash"
Ejecutamos de nuevo el script, comprobamos que se le han asignado permisos SUID a la /bin/bash y con el comando bash -p obtenemos una shell de root
 ![image](https://github.com/user-attachments/assets/f7e820aa-9c54-4196-8469-2c98318086d5)

Con esto ya tendríamos la maquina comprometida, en la que hemos aprendido como detectar y explotar un LFI, como realizar movimiento lateral a otro usuario explotando sudo, y por ultimo como escalar privilegios realizando un  Python Library Hijacking.
