CentOS 7 - WildFly 10
==================================================================================

### Pré-requisitos

1- Atualizar sistema

	yum -y update

2- Baixar "wget", ferramenta para download web.

	yum install -y wget

### Instalando e configurando Java

1- Baixar JDK confirmando a requisição dos termosde licença.

	wget --no-check-certificate -c --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u162-b12/0da788060d494f5095bf8624735fa2f1/jdk-8u162-linux-x64.tar.gz 

2- Extrair arquivo do JDK

	tar -zxvf jdk-8u162-linux-x64.tar.gz -C /opt/

3- Criar link simboliko para JDK

	ln -s /opt/jdk1.8.0_162 /opt/jdk

4- Instalar Java

	alternatives --install /usr/bin/java java /opt/jdk/bin/java 2
	alternatives --config java

5- Na saída do comando "alternatives --config java" escolha a versão mais recente 
do Java e pressione <ENTER>.

6- Configurar camaninho dos comandos "javac" e "jar" 

	alternatives --install /usr/bin/jar jar /opt/jdk/bin/jar 2
	alternatives --install /usr/bin/javac javac /opt/jdk/bin/javac 2
	alternatives --set jar /opt/jdk/bin/jar 
	alternatives --set javac /opt/jdk/bin/javac

7- Verificando configuração dos comandos java

	whereis java
	whereis jar
	whereis javac

8- Verificando versão do Java

	java -version

9- Criando scrpt para configuração de variaveis de ambiente para todos os usuários

	cat > /etc/profile.d/java.sh << "EOF"

		if ! echo ${PATH} | grep -q /opt/jdk/bin
		then
		   export PATH=/opt/jdk/bin:${PATH}
		fi

		if ! echo ${PATH} | grep -q /opt/jdk/jre/bin
		then
		   export PATH=/opt/jdk/jre/bin:${PATH}
		fi

		export JAVA_HOME=/opt/jdk
		export JRE_HOME=/opt/jdk/jre
		export CLASSPATH=.:/opt/jdk/lib/tools.jar:/opt/jdk/jre/lib/rt.jar

	EOF

10- Criar outro arquivo de configuração de variaveis de ambiente

	cat > /etc/profile.d/java.csh << "EOF"

		if ( "${path}" !~ */opt/jdk/bin* ) then
		   set path = ( /opt/jdk/bin $path )
		endif

		if ( "${path}" !~ */opt/jdk/jre/bin* ) then
		    set path = ( /opt/jdk/jre/bin $path )
		endif

		setenv JAVA_HOME /opt/jdk
		setenv JRE_HOME /opt/jdk/jre
		setenv CLASSPATH .:/opt/jdk/lib/tools.jar:/opt/jdk/jre/lib/rt.jar

	EOF

11- Alterar propietário e permissões dos arquivos criados

	chown root:root /etc/profile.d/java.sh
	chmod 755 /etc/profile.d/java.sh
	chown root:root /etc/profile.d/java.csh
	chmod 755 /etc/profile.d/java.csh

### Instalando e configurando WildFly

1- Baixar WildFly

	wget http://download.jboss.org/wildfly/10.0.0.Final/wildfly-10.0.0.Final.tar.gz

2- Extrair arquivo do WildFly

	tar -zxvf wildfly-10.0.0.Final.tar.gz -C /opt/

3- Criar link para Wildfly

	ln -s /opt/wildfly-10.0.0.Final /opt/wildfly

4- Criando usuário wildfly 

	useradd -s /sbin/nologin wildfly
	chown -R wildfly /opt/wildfly-10.0.0.Final
l
5- Adicionando linhas de definição do caminho das variáveis "JBOSS_HOME" e 
"JAVA_HOME" no arquivo "standlone.conf"

	echo "JBOSS_HOME=/opt/wildfly" >> /opt/wildfly/bin/standalone.conf
	echo "JAVA_HOME=/opt/jdk" >> /opt/wildfly/bin/standalone.conf

6- Você tabém pode definir a variável "JBOSS_HOME" adicionando a seguinte linha 
no arquivo "/etc/profile"

	nano /etc/profile

		JBOSS_HOME="/opt/wildfly"

7- Configurando "standalone.xml" para acessar o WildFly via rede. Localize as 
seguintes linhas e substituia o endereço "127.0.0.1" pelo seu endereço IP 
utilizado(192.168.56.102). 

	nano /opt/wildfly-10.1.0.Final/standalone/configuration/standalone.xml

		<wsdl-host>${jboss.bind.address:192.168.56.102}</wsdl-host>
		...
		<inet-address value="${jboss.bind.address.management:192.168.56.102}"/>
		...
		<inet-address value="${jboss.bind.address:192.168.56.102}"/>

8- Criar usuário de gerenciamento do WildFly via Web

	/opt/wildfly-10.1.0.Final/bin/add-user.sh

9- Segue abaixo a saida do comando acima

###########################################################################################################################################
What type of user do you wish to add?
 a) Management User (mgmt-users.properties)
 b) Application User (application-users.properties)
(a): a

Enter the details of the new user to add.
Using realm 'ManagementRealm' as discovered from the existing property files.
Username : admin
The username 'admin' is easy to guess
 Are you sure you want to add user 'admin' yes/no? yes
 Password recommendations are listed below. To modify these restrictions edit the add-user.properties configuration file.
 - The password should be different from the username
 - The password should not be one of the following restricted values {root, admin, administrator}
 - The password should contain at least 8 characters, 1 alphabetic character(s), 1 digit(s), 1 non-alphanumeric symbol(s)
Password :
Re-enter Password :
What groups do you want this user to belong to? (Please enter a comma separated list, or leave blank for none)[  ]: ManagementRealm
About to add user 'admin' for realm 'ManagementRealm'
Is this correct yes/no? yes
Added user 'admin' to file '/opt/wildfly-10.1.0.Final/standalone/configuration/mgmt-users.properties'
Added user 'admin' to file '/opt/wildfly-10.1.0.Final/domain/configuration/mgmt-users.properties'
Added user 'admin' with groups ManagementRealm to file '/opt/wildfly-10.1.0.Final/standalone/configuration/mgmt-groups.properties'
Added user 'admin' with groups ManagementRealm to file '/opt/wildfly-10.1.0.Final/domain/configuration/mgmt-groups.properties'
Is this new user going to be used for one AS process to connect to another AS process?
e.g. for a slave host controller connecting to the master or for a Remoting connection for server to server EJB calls.
yes/no? yes
To represent the user add the following to the server-identities definition <secret value="aGl0ZXNoQDEyMw==" />
		
################################################################################################################################################

16- Inicializar o serviço do WildFly

	/opt/wildfly/bin/standalone.sh -b 0.0.0.0 -bmanagement 0.0.0.0

### Iniciar junto com sistema

17- Criando Link simbolico do WildFly na inicialização do sistema

	ln -s 	/opt/wildfly-10.1.0.Final/docs/contrib/scripts/systemd/wildfly-init-redhat.sh /etc/systemd/system/

18- Criar Link simbolico dos arquivos de configuração do WildFly na pasta dos 
arquivos de configuração padrão do Linux

	ln -s /opt/wildfly-10.1.0.Final/docs/contrib/scripts/systemd/wildfly.conf /etc/default/

19- Alterar arquivo "/opt/wildfly-10.1.0.Final/docs/contrib/scripts/systemd/wildfly.conf", 
descomentando as linha abaixo:

	vi /opt/wildfly-10.1.0.Final/docs/contrib/scripts/systemd/wildfly.conf 

		JBOSS_USER=wildfly
		...
		JBOSS_OPTS="-b 0.0.0.0 -bmanagement=0.0.0.0"
		
20- Execute o wildfly

	service wildfly start

21- Fazer wildfly ser executado automaticamente na inicialização.

	chkconfig wildfly on

22- Liberando no Firewall

	firewall-cmd --zone=public --add-port=8080/tcp --permanent
	firewall-cmd --zone=public --add-port=9990/tcp --permanent
	firewall-cmd --reload


Instalação automatica(através de script)
==================================================================================

1- Baixar git

	yum install -y wget git nano 

	yum install -y httpd

	systemctl start httpd
	systemctl enable httpd

	firewall-cmd --permanent --add-port=80/tcp
	firewall-cmd --zone=public --add-port=8080/tcp --permanent
	firewall-cmd --zone=public --add-port=9990/tcp --permanent
	firewall-cmd --reload
	
2- Baixar JDK

	wget <URL_JDK>

	git clone https://gist.github.com/sukharevd/6087988/

	nano /6087988/wildfly-install.sh 

		#sed -i -e 's,<deployment-scanner path="deployments" relative-to="jboss.server.base.dir" scan-interval="5000",<deployment-scanner path="deployments" relative-to="jboss.server.base.dir" scan-interval="5000" deployment-timeout="'$WILDFLY_STARTUP_TIMEOUT'",g' $WILDFLY_DIR/$WILDFLY_MODE/configuration/$WILDFLY_MODE.xml
		sed -i -e 's,<inet-address value="${jboss.bind.address:127.0.0.1}"/>,<any-address/>,g' $WILDFLY_DIR/$WILDFLY_MODE/configuration/$WILDFLY_MODE.xml
		#sed -i -e 's,<socket-binding name="ajp" port="${jboss.ajp.port:8009}"/>,<socket-binding name="ajp" port="${jboss.ajp.port:28009}"/>,g' $WILDFLY_DIR/$WILDFLY_MODE/configuration/$WILDFLY_MODE.xml
		#sed -i -e 's,<socket-binding name="http" port="${jboss.http.port:8080}"/>,<socket-binding name="http" port="${jboss.http.port:28080}"/>,g' $WILDFLY_DIR/$WILDFLY_MODE/configuration/$WILDFLY_MODE.xml
		#sed -i -e 's,<socket-binding name="https" port="${jboss.https.port:8443}"/>,<socket-binding name="https" port="${jboss.https.port:28443}"/>,g' $WILDFLY_DIR/$WILDFLY_MODE/configuration/$WILDFLY_MODE.xml
		#sed -i -e 's,<socket-binding name="osgi-http" interface="management" port="8090"/>,<socket-binding name="osgi-http" interface="management" port="28090"/>,g' $WILDFLY_DIR/$WILDFLY_MODE/configuration/$WILDFLY_MODE.xml

	cp 6087988/wildfly-install.sh /opt/

	chmod +x /opt/wildfly-install.sh

	/opt/wildfly-install.sh

### Configurando "standalone.xml" para acessar o WildFly via rede. Localize as 
seguintes linhas e substituia o endereço "127.0.0.1" pelo seu endereço IP 
utilizado(192.168.56.102). 

	nano /opt/wildfly-10.1.0.Final/standalone/configuration/standalone.xml

		<wsdl-host>${jboss.bind.address:192.168.56.102}</wsdl-host>
		...
		<inet-address value="${jboss.bind.address.management:192.168.56.102}"/>
		...
		<inet-address value="${jboss.bind.address:192.168.56.102}"/>
		
#

	/opt/wildfly/bin/add-user.sh

	systemctl restart wildfly
	systemctl enable wildfly
		
### Instalação de Mysql JDBC-DRIVER no Wildfly

1- Fazer download do driver: Mysql JDBC

2- Criar subpastas "/opt/wildfly/modules/system/layers/base/com/mysql/main/", ou "/opt/wildfly/modules/com/mysql/main/".

3- Copiar o arquivo "mysql-connector-java-5.1.46-bin.jar" para dentro da pasta criada.

4- Criar arquivo "module.xml" no "$WILDFLY_HOME/modules/system/layers/base/com/mysql/main" com seguinte conteúdo:

	<?xml version="1.0" encoding="UTF-8"?>
	<module xmlns="urn:jboss:module:1.1" name="com.mysql">
  		<resources>
    			<resource-root path="mysql-connector-java-5.1.46-bin.jar"/>
  		</resources>
  		<dependencies>
    			<module name="javax.api"/>
    			<module name="javax.transaction.api"/>
  		</dependencies>
	</module>

5- Adicionar a seguinte linha no "standalone.xml" dentro da tag "drivers".

	<driver name="mysql" module="com.mysql">
 		<driver-class>com.mysql.jdbc.jdbc2.optional.MysqlDataSource</driver-class>
	</driver>

6- Opcionalmente à instrução acima você pode executar o seguinte comando.

	./wildfly/bin/jboss-cli.sh --connect /subsystem=datasources/jdbc-driver=mysql:add'(driver-name=mysql,driver-module-name=com.mysql,driver-class-name=com.mysql.jdbc.jdbc2.optional.MysqlDataSource)'


7- adicionar a seguinte linha no "standalone.xml" dentro da tag "drivers".
Para adicionar um XA-Datasources.

		<driver name="mysql" module="com.mysql">
      		<xa-datasource-class>com.mysql.jdbc.jdbc2.optional.MysqlXADataSource</xa-datasource-class>
    	</driver>


8- Opcionalmente à instrução acima você pode executar o seguinte comando.

	./wildfly/bin/jboss-cli.sh --connect /subsystem=datasources/jdbc-driver=mysql:add'(driver-name=mysql,driver-module-name=com.mysql,driver-xa-datasource-class-name=com.mysql.jdbc.jdbc2.optional.MysqlXADataSource)'


9- Adicione no arquivo "standalone.xml" uma definição de fonte de dados dentro da tag "datasources".


	<datasource jndi-name="java:jboss/datasources/java" pool-name="java" enabled="true" use-java-context="true" use-ccm="true">
 		<connection-url>jdbc:mysql://[HOST_NAME]:3306:[Database]</connection-url>
  		<driver>mysql[has to match the driver name]</driver>
  		<pool>
   			<min-pool-size>1</min-pool-size>
   			<max-pool-size>5</max-pool-size>
   			<prefill>true</prefill>
  		</pool>
  		<security>
   			<user-name>[USER]</user-name>
   			<password>[PWD]</password>
  		</security>
	</datasource>

### Instalção de Oracle JDBC-DRIVER no Wildfly

1- Fazer download do driver: OJDBC

2- Criar subpastas "/opt/wildfly/modules/system/layers/base/com/oracle/main/", ou "/opt/wildfly/modules/com/oracle/main/.

3- Copiar o arquivo "ojdc[VERSION].jar" para dentro da pasta criada.

4- Criar arquivo "module.xml" no "$WILDFLY_HOME/modules/system/layers/base/com/oracle/main" com seguinte conteúdo:

	<?xml version="1.0" encoding="UTF-8"?>
	<module xmlns="urn:jboss:module:1.1" name="com.oracle">
  		<resources>
    			<resource-root path="ojdbc[VERSION].jar"/>
  		</resources>
  		<dependencies>
    			<module name="javax.api"/>
    			<module name="javax.transaction.api"/>
  		</dependencies>
	</module>

5- Adicionar a seguinte linha no "standalone.xml" dentro da tag "drivers".

	<driver name="oracle" module="com.oracle">
 		<driver-class>oracle.jdbc.driver.OracleDriver</driver-class>
	</driver>

6- Opcionalmente à instrução acima você pode executar o seguinte comando.

	./wildfly/bin/jboss-cli.sh --connect /subsystem=datasources/jdbc-driver=oracle:add'(driver-name=oracle,driver-module-name=com.oracle,driver-class-name=oracle.jdbc.driver.OracleDriver)'

7- adicionar a seguinte linha no "standalone.xml" dentro da tag "drivers".
Para adicionar um XA-Datasources.

		<driver name="oracle" module="com.oracle">
      		<xa-datasource-class>oracle.jdbc.xa.client.OracleXADataSource</xa-datasource-class>
    	</driver>


8- Opcionalmente à instrução acima você pode executar o seguinte comando.

	./wildfly/bin/jboss-cli.sh --connect /subsystem=datasources/jdbc-driver=oracle:add'(driver-name=oracle,driver-module-name=com.oracle,driver-xa-datasource-class-name=oracle.jdbc.xa.client.OracleXADataSource)'

9- Adicione no arquivo "standalone.xml" uma definição de fonte de dados dentro da tag "datasources".


	<datasource jndi-name="java:/[NAME]" pool-name="OracleDS" enabled="true">
 		<connection-url>jdbc:oracle:thin:@[HOST_NAME]:1521:[SID]</connection-url>
  		<driver>oracle[has to match the driver name]</driver>
  		<pool>
   			<min-pool-size>1</min-pool-size>
   			<max-pool-size>5</max-pool-size>
   			<prefill>true</prefill>
  		</pool>
  		<security>
   			<user-name>[USER]</user-name>
   			<password>[PWD]</password>
  		</security>
	</datasource>

## Observações ao JDBC 

1- em alguns casos é necessário excluir todo o Wilfly 
e descompactar um novo, pois algumas configurações como a 
adição de driver de banco de dados acaba se tornando 
fixa, ou bugando a inserção de um driver. Alguns dos 
motivos para isso ocorrer seria o fato de utilizar o
"Jboss-cli.sh" para configurar alguns sistemas.


### JBOSS-CLI

1- Descobrir quais arquivos .war são implementados:

	./bin/jboss-cli.sh --connect --commands=ls\ deployment | grep war

2- Para uma lista de operações específicas (por exemplo, operações relacionadas ao subsistema de registro), você sempre pode consultar o modelo em si. Por exemplo, para ler as operações suportadas pelo recurso do subsistema de criação de log em um servidor independente:
	
	./bin/jboss-cli.sh --connect /subsystem=logging:read-operation-names

### Usando interface Web

