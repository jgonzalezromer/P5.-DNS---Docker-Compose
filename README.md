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
Agora falaremos en como se divide este proyecto, onde teremos por unha parte o arquivo principal o cal chamaremos `docker-compose.yml`, este é un arquivo de configuración que define e executa aplicacions multi-contenedor de Docker, nel especificamos como se debe construir e executar os contenedores. Por outra parte teremos dous directorios principais como manda a [setup da imaxe](https://hub.docker.com/r/internetsystemsconsortium/bind9).
