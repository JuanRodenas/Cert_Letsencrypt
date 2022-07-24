# Cert_Letsencrypt

#### Crear una cuenta en:
[DuckDNS](https://www.duckdns.org)
- Copie el token de su dominio y el dominio creado.

![alt text](https://github.com/JuanRodenas/Cert_Letsencrypt/blob/main/token-duckdns.png)

![alt text](https://github.com/JuanRodenas/Cert_Letsencrypt/blob/main/Dominio-duckdns.png)


#### Descargar el repositorio
```
git clone https://github.com/JuanRodenas/Cert_Letsencrypt.git
```

#### editamos los parámetros de nuestro dominio y correo electrónico.
```
- URL=YOUR_DOMAIN.duckdns.org
- DUCKDNSTOKEN=YOUR_TOKEN
- EMAIL="YOUR_MAIL"
```

#### Desplegamos el contenedor docker
```
docker-compose up -d
```
#### Comprobamos los archivos creados en el directorio
```
$ ls -l /root/docker/letsencrypt/config/etc/letsencrypt/archive/YOUR_DOMAIN.duckdns.org/
total 20
-rw-r--r-- 1 root root 2208 feb 15 11:13 certX.pem
-rw-r--r-- 1 root root 3749 feb 15 11:13 chainX.pem
-rw-r--r-- 1 root root 5957 feb 15 11:13 fullchainX.pem
-rw------- 1 root root 3272 feb 15 11:13 privkeyX.pem
```
Tendremos el certificado con su clave privada y el certificado intermedio, elegiremos archivo, donde X es el número de archivo más alto que tengamos, los anteriores no se borran.
