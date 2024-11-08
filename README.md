# P6-Configuraci-n-CLientes-Servidor-DNS

Primeramente comenzaremos por crear y configurar nuestro archivo yml para poder configurar las redes que vamos a necesitar nuestro Cliente-Servidor.El código que usaremos es el siguiente:
```
version: '3'

services:
  asir_bind9:
    container_name: asir_bind9
    image: ubuntu/bind9
    platform: linux/amd64
    ports:
      - "53:53"  # Exponer el puerto 53 para que otros puedan hacer consultas DNS
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.1  # IP estática del servidor DNS
    volumes:
      - ./conf:/etc/bind/  # Montar la configuración del servidor BIND
      - ./zonas:/var/lib/bind/  # Montar el directorio de zonas
    restart: unless-stopped  # Reiniciar el contenedor si falla o si Docker se reinicia

  cliente:
    container_name: P6_apline
    image: alpine
    platform: linux/amd64
    tty: true
    stdin_open: true
    dns:
      - 172.28.5.1  # El cliente usará el DNS del servidor asir_bind9
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.2  # IP estática del cliente en la misma subred
    restart: unless-stopped  # Asegurarse de que el cliente también se reinicie si falla

networks:
  bind9_subnet:
    driver: bridge  # Tipo de red 'bridge'
    ipam:
     
      config:
        - subnet: "172.28.0.0/16"  # Definir el rango de subred
          ip_range: "172.28.5.0/24"  # Rango de IPs para los contenedores
          gateway: "172.28.5.254"  # Puerta de enlace de la red
          ```
```
**Para este código yml lo que utilizamos fue la base que ya estaba creada y subida en el github de damián lo que hice yo aparte fue otro archivo yml aparte con la red creada dentro de el y con ayuda de chat gpt le pedí que me juntara estos dos archivos.**

**Agora temos que crear e copiar as carpetas zoas e conf dentro do directorio, porque si non os servers non se configuraran e logo zoas sen este arquivo non podremos facer dig mais adiante para poder comprobar que todo esta feito correctamente.**

Una vez configurado el archivo lo que haremos sera levantar los contenedores que tenemos configurados para ello utilizaremos el siguiente comando.
```
docker compose up
```
Se puede añadir el -d para que libere la terminal pero para mi es mas cómodo y más facil de visualizar al tener otra terminal en ejecución. 

**Una vez con los contenedores accederemos a nuestro contenedor cliente que en este caso es: "P6_apline", para ello utilizaremos:**
```
docker exec -it P6_apline /bin/sh
```
Una vez dentro para comprobar que tenemos conexión entre los servidores primero deberemos de seguir una seria de pasos y tendremos que descargar ciertas herramientas para poder hacer un DIG dentro de la máquina.

**Para ello utilizaremos los siguientes comandos por orden:**
```
apk update
apk add bind-tools
```

Con estos dos comandos non debería de haber ningún problema para poder realizar o dig.

**Como ultimo para comprobar o dig facemos dig a @172.28.5.1 test.asircastelao.int**
E a miña resposta foi a seguinte: 
```
**test.asircastelao.int.	38400	IN	A	172.28.5.4**
``
