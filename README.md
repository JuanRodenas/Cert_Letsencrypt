## Certificados para dominios de Duckdns con _Let's encrypt_
<p align="center">
  <img src="https://github.com/JuanRodenas/Cert_Letsencrypt/blob/main/duckdns-letsencrypt.png"
       width="700"/>
</p>

<p align="center">
  <img src="https://logos-world.net/wp-content/uploads/2021/02/Docker-Symbol.png" 
       width="300"/>
</p>

### Crear una cuenta en:
[DuckDNS](https://www.duckdns.org)
- Copie el token de su dominio y el dominio creado.

![alt text](https://github.com/JuanRodenas/Cert_Letsencrypt/blob/main/token-duckdns.png)

![alt text](https://github.com/JuanRodenas/Cert_Letsencrypt/blob/main/Dominio-duckdns.png)


### Descargar el repositorio
```
git clone https://github.com/JuanRodenas/Cert_Letsencrypt.git
```
### Arquitecturas soportadas
Soporta múltiples arquitecturas como `x86-64`, `arm64` and `armhf`. Las arquitecturas soportadas por esta imagen son:

| Architecture | Tag |
| :----: | --- |
| x86-64 | amd64-latest |
| arm64 | arm64v8-latest |
| armhf | arm32v7-latest |

### Parameters
| Parameter | Function |
| :----: | --- |
| `-p 443` | Https port |
| `-p 80` | Http port (required for http validation and http -> https redirect) |
| `-e PUID=1000` | for UserID - see below for explanation |
| `-e PGID=1000` | for GroupID - see below for explanation |
| `-e TZ=Europe/London` | Specify a timezone to use EG Europe/London. |
| `-e URL=yourdomain.url` | Top url you have control over (`customdomain.com` if you own it, or `customsubdomain.ddnsprovider.com` if dynamic dns). |
| `-e VALIDATION=http` | Certbot validation method to use, options are `http`, `dns` or `duckdns` (`dns` method also requires `DNSPLUGIN` variable set) (`duckdns` method requires `DUCKDNSTOKEN` variable set, and the `SUBDOMAINS` variable must be either empty or set to `wildcard`). |
| `-e SUBDOMAINS=www,` | Subdomains you'd like the cert to cover (comma separated, no spaces) ie. `www,ftp,cloud`. For a wildcard cert, set this _exactly_ to `wildcard` (wildcard cert is available via `dns` and `duckdns` validation only) |
| `-e CERTPROVIDER=` | Optionally define the cert provider. Set to `zerossl` for ZeroSSL certs (requires existing [ZeroSSL account](https://app.zerossl.com/signup) and the e-mail address entered in `EMAIL` env var). Otherwise defaults to Let's Encrypt. |
| `-e DNSPLUGIN=cloudflare` | Required if `VALIDATION` is set to `dns`. Options are `aliyun`, `cloudflare`, `cloudxns`, `cpanel`, `desec`, `digitalocean`, `directadmin`, `dnsimple`, `dnsmadeeasy`, `dnspod`, `domeneshop`, `gandi`, `gehirn`, `google`, `he`, `hetzner`, `infomaniak`, `inwx`, `ionos`, `linode`, `luadns`, `netcup`, `njalla`, `nsone`, `ovh`, `rfc2136`, `route53`, `sakuracloud`, `standalone`, `transip` and `vultr`. Also need to enter the credentials into the corresponding ini (or json for some plugins) file under `/config/dns-conf`. |
| `-e PROPAGATION=` | Optionally override (in seconds) the default propagation time for the dns plugins. |
| `-e DUCKDNSTOKEN=` | Required if `VALIDATION` is set to `duckdns`. Retrieve your token from [DuckDNS](https://www.duckdns.org/) |
| `-e EMAIL=` | Optional e-mail address used for cert expiration notifications (Required for ZeroSSL). |
| `-e ONLY_SUBDOMAINS=false` | If you wish to get certs only for certain subdomains, but not the main domain (main domain may be hosted on another machine and cannot be validated), set this to `true` |
| `-e EXTRA_DOMAINS=` | Additional fully qualified domain names (comma separated, no spaces) ie. `extradomain.com,subdomain.anotherdomain.org,*.anotherdomain.org` |
| `-e STAGING=false` | Set to `true` to retrieve certs in staging mode. Rate limits will be much higher, but the resulting cert will not pass the browser's security test. Only to be used for testing purposes. |
| `-v /config` | All the config files including the webroot reside here. |

### editamos los parámetros de nuestro dominio y correo electrónico.
```
- URL=YOUR_DOMAIN.duckdns.org
- DUCKDNSTOKEN=YOUR_TOKEN
- EMAIL="YOUR_MAIL"
```

### Desplegamos el contenedor docker
```
docker-compose up -d
```
### Comprobamos los archivos creados en el directorio
```
$ ls -l /root/docker/letsencrypt/config/etc/letsencrypt/archive/YOUR_DOMAIN.duckdns.org/
total 20
-rw-r--r-- 1 root root 2208 feb 15 11:13 certX.pem
-rw-r--r-- 1 root root 3749 feb 15 11:13 chainX.pem
-rw-r--r-- 1 root root 5957 feb 15 11:13 fullchainX.pem
-rw------- 1 root root 3272 feb 15 11:13 privkeyX.pem
```
Tendremos el certificado con su clave privada y el certificado intermedio, elegiremos archivo, donde X es el número de archivo más alto que tengamos, los anteriores no se borran.

## Sacar el .crt y .key de un .pfx

Si tenemos un certificado en formato .pfx, y lo que necesitamos es el .crt y la clave privada en .key, tenemos que separar dichos ficheros del .pfx
Para ello, solo necesitamos OpenSSL, y la clave PEM con la que se genero el .pfx\

![alt text](https://github.com/JuanRodenas/Cert_Letsencrypt/blob/main/img_cert.png)

**Extraer la clave privada a un fichero .key:**
```
openssl pkcs12 -in TU_FICHERO.pfx -nocerts -out FICHERO_CLAVE_PRIVADA.key
```

nos pedirá nuestra clave PEM, la metemos, nos la pedirá un par de veces para confirmar y si ha ido todo correctamente, tendremos nuetro .key con la clave privada

**Extraer el certificado a un fichero .crt:**
```
openssl pkcs12 -in TU_FICHERO.pfx -clcerts -nokeys -out CERTIFICADO.crt
```
lo mismo de antes, nos pedirá la clave PEM, y tendremos el .crt que es nuestro certificado.

**Pero que ocurre si lo que necesitamos es nuestra clave privada sin cifrar?
```
openssl rsa -in FICHERO_CLAVE_PRIVADA.key -out FICHERO_CLAVE_PRIVADA_SIN_CIFRAR.key
```

Nos volverá a pedir la clave PEM que nos pidio la primera vez que sacamos el .key

También podemos tener nuestra clave privada, en formato PEM.
```
openssl rsa -in FICHERO_CLAVE_PRIVADA.key -outform PEM -out FICHERO_CLAVE_PRIVADA_PEM.key
```
