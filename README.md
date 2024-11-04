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
mkdir P5_DNS_Docker_Compose
```
Agora falaremos en como se divide este proyecto, onde teremos por unha parte o arquivo principal o cal chamaremos `docker-compose.yml`, este é un arquivo de configuración que define e executa aplicacions multi-contenedor de Docker, nel especificamos como se debe construir e executar os contenedores. 

Por outra parte teremos dous directorios principais que serviran como volumes como manda a [setup da imaxe](https://hub.docker.com/r/internetsystemsconsortium/bind9). Os cales serán `/etc/bind` e `/var/cache/bind`, os cales podremos crear rapidamente usando o comando:
```
mkdir -p /etc/bind /var/cache/bind
```
A rama de directorios de `/etc` serven para a configuración de BIND e a rama de `/var` serve para o espazo de traballo do funcionamento de BIND.

---
Antes de pasar a crear os documentos de configuración faremos o comando:
```
sudo chown -R 100:100 ./etc/bind/ ./var/cache/bind
```
O cal fai que os donos dos directorios e subdirectores sinalados cambien a o propietario e grupo 100.

Isto xa que asi nos aseguramos que BIND teña os permisos necesarios para funcionar dentro dun contenedor de Docker, o cal é clave para o funcionamento e a seguridade.
