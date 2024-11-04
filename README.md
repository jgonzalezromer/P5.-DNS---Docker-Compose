<h1>
<p align=center>
P5-DNS e Docker Compose 
</p>
</h1>
<h3>
<p align=center>
Juan Gabriel González Romero
</p>
</h3>

---
### Proposta: Configuración da imaxe bind9 utilizando como medio Docker-Compose
---
### Organización
Para comezar crearemos unha carpeta onde atoparase todo o noso proyecto:
```
mkdir P5_DNS_Docker_Compose && cd P5_DNS_Docker_Compose
```
Agora falaremos en como se divide este proyecto, onde teremos por unha parte o arquivo principal o cal chamaremos `docker-compose.yml`, este é un arquivo de configuración que define e executa aplicacions multi-contenedor de Docker, nel especificamos como se debe construir e executar os contenedores. 

Por outra parte teremos dous directorios principais que serviran como volumes como manda a [setup da imaxe](https://hub.docker.com/r/internetsystemsconsortium/bind9). Os cales serán `/etc/bind` e `/var/cache/bind`, os cales podremos crear rapidamente usando o comando:
```
mkdir -p /etc/bind /var/cache/bind
```
A rama de directorios de `/etc` serven para a configuración de BIND e a rama de `/var` serve para o espazo de traballo do funcionamento de BIND.

---
### Permisos
Antes de pasar a crear os documentos de configuración faremos o comando:
```
sudo chown -R 100:100 ./etc/bind/ ./var/cache/bind
```
O cal fai que os donos dos directorios e subdirectores sinalados cambien a o propietario e grupo 100.

Isto xa que asi nos aseguramos que BIND teña os permisos necesarios para funcionar dentro dun contenedor de Docker, o cal é clave para o funcionamento e a seguridade.

---
### docker-compose.yml
Crearemos este arquivo no directorio `P5_DNS_Docker_Compose`:
```
nano ./docker-compose.yml
```
Nel escribiremos o codigo o cal colleremos como referencia o que pon na [setup da imaxe](https://hub.docker.com/r/internetsystemsconsortium/bind9):
```
BIND 9.18 (extended support version, aka 'old stable')

docker run \
        --name=bind9 \
        --restart=always \
        --publish 53:53/udp \
        --publish 53:53/tcp \
        --publish 127.0.0.1:953:953/tcp \
        internetsystemsconsortium/bind9:9.18
```
Nel podemos distinguir os portos que podemos utilizar:
```
53:53/udp
53:53/tcp 
127.0.0.1:953:953/tcp 
```
Aunque o porto 53 pode dar fallo, se pasa iso utilizaremos:
```
54:53/udp
54:53/tcp 
127.0.0.1:953:953/tcp 
```
Por outra parte temos a imaxe de BIND que utilizaremos:
```
internetsystemsconsortium/bind9:9.18
```
E a opción de `restart=always`.

Isto o pasaremos a yml añadindo os volumes que vimos antes e quedaranos o código:
```
services:
  #Servizos que se executarán no contenedor
  bind9: #Como queremos chamar nos o servizo
    image: internetsystemsconsortium/bind9:9.18 #Imaxe que utilizaremos no contenedor
    container_name: Practica5_bind9 #Nome específico do contenedor
    ports:
      #Mapeo dos portos host:contenedor
      - 54:53/udp
      - 54:53/tcp
      - 127.0.0.1:953:953/tcp #Neste só pertmite conexións desde localhost
    volumes:
      #Volumes onde se montará o contenedor
      - ./etc/bind:/etc/bind
      - ./var/cache/bind:/var/cache/bind
    restart: always #Esta opción indica que ó contenedor debe reiniciarse en ciertas condiciones
```
Nel temos que ter en conta que nos volumes primeiro escribimos a ruta que terán no host `./etc/bind` e logo escribiremos a ruta no sistema de archivos do contenedor `/etc/bind`. Tamén hai que ter coidado coa indentacion xa que temos que utilizar 2 espacios en vez de tabular.

---
### named.conf
Este arquivo é crucial para configurar el servidor BIND, este conten a configuración principal do servidor. Neste caso facendo caso a [setup da imaxe](https://hub.docker.com/r/internetsystemsconsortium/bind9) escribiremos dentro:
```
options {
    directory "/var/cache/bind";
};
```
Isto fai que o servidor sepa que ten que almacenar os seus arquivos de caché e outros datos no directorio `/var/cache/bind`, como xa falamos antes está ruta ponse de unha forma absoluta porque refirese a ruta que terá o contenedor.

---
### Comprobación
Agora toca o momento de comprobar que todo funcionou, poñemonos no directorio onde temos o `docker-compose.yml` e lanzamos desde a terminal o comando:
```
docker compose up -d #O -d serve para que se execute en segundo plano
```
E debería saltar o seguinte mensaxe:
```
[+] Running 2/2
 ✔ Network <nome_da_network>  Created                                      0.1s 
 ✔ Container Practica5_bind9  Started                                      0.8s 
```
Agora temos que saber cal é a IP do container polo que faremos o comando:
```
docker network inspect <nome_da_network>
```
Buscaremos na resposta o apartado `Containers` e a liña de `IPv4Address`:
```
"Containers": {
            "0512ea52dde7393d1a762a393c78cea5688dbc6c7b5d0bcbcadb826aad59ed7b": {
                "Name": "Practica5_bind9",
                "EndpointID": "1ca7b6de3e6355e1c9aa78c60c1c6ffd3e4b1ac67a5b989314de2be5c4247c2d",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
```
Agora coa IP faremos o comando:
```
dig @172.18.0.2
```
Isto xa que co comando `dig` consultaremos directamente o servidor DNS e sabremos se está funcionando correctamente, a resposta que nos pode chegar sería algo asi:
```
; <<>> DiG 9.18.28-0ubuntu0.20.04.1-Ubuntu <<>> @172.18.0.2
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16033
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 27

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 75697c8ea2663a47010000006728cb0952143d20e9bf8053 (good)
;; QUESTION SECTION:
;.				IN	NS

;; ANSWER SECTION:
.			517912	IN	NS	d.root-servers.net.
.			517912	IN	NS	g.root-servers.net.

```

---
### Errores comúns
