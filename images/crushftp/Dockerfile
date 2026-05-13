FROM eclipse-temurin:21-jre-jammy

RUN apt-get update && apt-get install -y \
  bash \
  ca-certificates \
  curl \
  wget \
  unzip \
  procps \
  && update-ca-certificates \
  && rm -rf /var/lib/apt/lists/*

# Setup CrushFTP
RUN wget -O /tmp/CrushFTP11.zip https://www.crushftp.com/early11/CrushFTP11.zip
RUN unzip -o -q /tmp/CrushFTP11.zip -d /var/opt
RUN rm -f /tmp/CrushFTP11.zip
RUN chmod +x /var/opt/CrushFTP11/crushftp_init.sh

# Set prefs.XML to persisted folder, and increase max JVM memory
RUN sed -i "s/-Xmx512M -jar plugins\/lib\/CrushFTPJarProxy\.jar/-Xmx2G -Dcrushftp.prefs=\/var\/opt\/CrushFTP11\/settings\/ -jar plugins\/lib\/CrushFTPJarProxy.jar/g" /var/opt/CrushFTP11/crushftp_init.sh

# Add SQLJBC for MSSQL
RUN wget -O /var/opt/CrushFTP11/plugins/lib/mssql-jdbc-12.8.1.jre11.jar \
  https://repo1.maven.org/maven2/com/microsoft/sqlserver/mssql-jdbc/12.8.1.jre11/mssql-jdbc-12.8.1.jre11.jar

ADD ./startup.sh /var/opt/startup.sh
ENTRYPOINT [ "/bin/bash", "/var/opt/startup.sh" ]
CMD ["-c"]

HEALTHCHECK --interval=1m --timeout=3s \
  CMD curl -f ${CRUSH_ADMIN_PROTOCOL}://localhost:${CRUSH_ADMIN_PORT}/favicon.ico -H 'Connection: close' || exit 1

ENV CRUSH_ADMIN_PROTOCOL http
ENV CRUSH_ADMIN_PORT 8080

EXPOSE 21 443 2000-2100 2222 8080 9090