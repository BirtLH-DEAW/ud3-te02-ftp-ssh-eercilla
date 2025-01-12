# Utilizamos una imagen oficial de Ubuntu
FROM ubuntu:18.04

# Damos información sobre la imagen que estamos creando
LABEL \
    version="1.0" \
    description="Ubuntu + Apache2 + virtual host" \
    maintainer="usuarioBIRT <usuarioBIRT@birt.eus>"

# Actualizamos la lista de paquetes e instalamos nano, apache2 y el resto de servicios
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y nano apache2 proftpd ssh git supervisor && \
    rm -rf /var/lib/apt/lists/*

# Creamos directorios para los sitios web y configuraciones
RUN mkdir -p /var/www/html/sitioPrimero /var/www/html/sitioSegundo

# Copiamos archivos al contenedor
COPY indexPrimero.html indexSegundo.html sitioPrimero.conf sitioSegundo.conf sitioPrimero.key SitioPrimero.cer /

# Archivos de configuración de proftpd
COPY 	proftpd.conf /etc/proftpd/proftpd.conf
COPY 	tls.conf /etc/proftpd/tls.conf
COPY 	modules.conf /etc/proftpd/modules.conf
COPY 	ftpusers /etc/ftpusers

# Archivos de TLS
COPY 	proftpd.crt /etc/ssl/certs/proftpd.crt
COPY 	proftpd.key /etc/ssl/private/proftpd.key

# Archivos de ssh
COPY sshd_config /etc/ssh/sshd_config


# Movemos los archivos a sus ubicaciones adecuadas
RUN mv /indexPrimero.html /var/www/html/sitioPrimero/index.html && \
    mv /indexSegundo.html /var/www/html/sitioSegundo/index.html && \
    mv /sitioPrimero.conf /etc/apache2/sites-available/sitioPrimero.conf && \
    mv /sitioSegundo.conf /etc/apache2/sites-available/sitioSegundo.conf && \
    mv /sitioPrimero.key /etc/ssl/private/sitioPrimero.key && \
    mv /SitioPrimero.cer /etc/ssl/certs/SitioPrimero.cer

# Habilitamos los sitios y el módulo SSL
RUN a2ensite sitioPrimero.conf && \
    a2ensite sitioSegundo.conf && \
    a2enmod ssl

# Creamos los usuarios que nos pide la tarea
RUN useradd -d /var/www/html/sitioPrimero -s /sbin/nologin eercilla1 && \
	echo "eercilla1:deaw" | chpasswd && \
	chown -R eercilla1:eercilla1 /var/www/html/sitioPrimero && \
	chmod -R 755 /var/www/html/sitioPrimero

RUN useradd -d /var/www/html/sitioSegundo -s /bin/bash eercilla2 && \
	echo "eercilla2:deaw" | chpasswd && \
	chown -R eercilla2:eercilla2 /var/www/html/sitioSegundo && \
	chmod -R 755 /var/www/html/sitioSegundo

RUN useradd -d /srv/ftp -s /bin/bash eercilla && \
	echo "eercilla:deaw" | chpasswd

# Crear el directorio /var/run/sshd y establecer permisos
RUN mkdir -p /var/run/sshd && \
    chmod 0755 /var/run/sshd && \
    chown root:root /var/run/sshd


# Crear un script para iniciar los servicios
RUN echo '#!/bin/bash\n\
    service apache2 start\n\
    service proftpd start\n\
    service ssh start\n\
    tail -f /dev/null' > /start_services.sh && chmod +x /start_services.sh

# Configurar SSH para que confíe en GitHub
RUN mkdir -p /root/.ssh && ssh-keyscan github.com >> /root/.ssh/known_hosts

# Clave para poder clonar el contenido git
COPY id_rsa.txt /root/.ssh/id_rsa_deaw.txt
RUN chmod 700 /root/.ssh/id_rsa_deaw.txt


# Iniciar el agente SSH y añadir la clave privada
RUN eval "$(ssh-agent -s)" && ssh-add /root/.ssh/id_rsa_deaw.txt
# Clonar el repositorio privado de GitHub
RUN GIT_SSH_COMMAND="ssh -i /root/.ssh/id_rsa_deaw.txt" git clone git@github.com:deaw-birt/UD3-ftp_anonimo.git /srv/ftp/deawTE03_02

# Exponemos los puertos
EXPOSE 80
EXPOSE 443
EXPOSE 21
EXPOSE 1022

# Iniciamos una serie de servicios al iniciar el contenedor
CMD ["/start_services.sh"]
