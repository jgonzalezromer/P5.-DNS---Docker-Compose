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
      - 127.0.0.1:953:953/tcp #Neste só pertmite conexións desde local host
    volumes:
      #Volumes onde se montará o contenedor
      - ./etc/bind:/etc/bind
      - ./var/cache/bind:/var/cache/bind
    restart: always #Esta opción indica que ó contenedor debe reiniciarse en ciertas condiciones
```
Nel temos que ter en conta que nos volumes primeiro escribimos a ruta que terán no host `./etc/bind` e logo escribiremos a ruta no sistema de archivos do contenedor `/etc/bind`. Tamén hai que ter coidado coa indentacion xa que temos que utilizar 2 espacios en vez de tabular.

---




