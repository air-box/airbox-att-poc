# AirBox - AT&T - Proof of Concept

It includes software, code, steps and evaluations of AirBox solution in RHEL8 environment using Apache tomcat server.
This PoC demo requires 3 machines (virtual machines, containers or baremetal servers): 
1. Test front end app / tomcat server
2. Test backend app / db server
3. AirBox KeyCentral server

The above setup is based on the model architecture that can be used to evaluate AirBox.

## Install Tomcat w/ AirBox

* There are no code level changes in the frontend tomcat server to use AirBox. 

* Steps to deploy tomcat on RHEL

  * Install Open-jdk-11
  ```
  $ sudo dnf update
  $ sudo dnf install java-11-openjdk java-11-openjdk-devel
  ```
  * Add Tomcat group
  ```$ sudo groupadd tomcat```

  * Create Tomcat directory
  ```$ sudo mkdir /opt/tomcat```

  * Create tomcat user, disable login and give rights
  ```$ sudo useradd -s /bin/nologin -g tomcat -d /opt/tomcat tomcat```

  * Download tomcat binary package
  ```
  $ sudo dnf install wget
  $ sudo wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.0.21/bin/apache-tomcat-10.0.21.tar.gz
  ```
  
  * Untar tomcat
  ```sudo tar -xvf apache-tomcat-10.0.21.tar.gz -C /opt/tomcat --strip-components=1```

  * Set permissions
  ```
  sudo chown -R tomcat: /opt/tomcat
  sudo sh -c 'chmod +x /opt/tomcat/bin/*.sh'
  ```

  * NOTE: Typically after this step you setup a tomcat service by creating tomcat.service file and enable it using systemctl.
  Ensure that service mode is off for purposes of this PoC demo.

* Setup tomcat-native
  * Install Openssl with headers
  ```$ dnf Install openssl-level```
  * Install apr with headers
  ```$ dnf Install apr-devel```
  * Install build essentials
  ```$ dnf group install "Development Tools"```
  * Download and install tomcat-native library
  ```
  $ mkdir /opt/tomcat-native
  $ wget https://dlcdn.apache.org/tomcat/tomcat-connectors/native/1.2.33/source/tomcat-native-1.2.33-src.tar.gz
  $ tar -xvf tomcat-native-1.2.33-src.tar.gz -C /opt/tomcat-native
  $ ./configure configure —with-apr=/usr/bin/apr-1-config —with-ssl=yes prefix=$CATALINA_HOME
  ```
  * Configure tomcat to use it
  $ Create or edit $CATALINA_HOME/bin/setenv.sh to add tomcat native to LD_LIBRARY_PATH
  ```
  CATALINA_HOME=/opt/tomcat
  LD_LIBRARY_PATH=/opt/air-box/:$LD_LIBRARY_PATH:$CATALINA_HOME/lib
  export LD_LIBRARY_PATH
  ```
 * Configure tomcat to handle HTTPS connections
  * Create and add SSL certs and keys and out them in the directory /opt/tomcat/ssl/ 
  * Add HTTPS APR connector in $CATALINA_HOME/conf/server.xml
  ```
  <Connector
    	protocol="org.apache.coyote.http11.Http11AprProtocol"
    	port="8443"
    	maxThreads="1"
    	SSLEnabled="true" >
 	<SSLHostConfig>
    		<Certificate
        	certificateKeyFile="ssl/server.key"
        	certificateFile="ssl/server.crt"
        	type="RSA"
        	/>
  	</SSLHostConfig>
    </Connector>
    ````
  ```

* AirBox can be installed by following the steps below:
    * Log in to the test app server:
    * Clone this repo 
    ````git clone https://github.com/air-box/airbox-att-poc.git```
    * Create AirBox directory
    ```mkdir -p /opt/air-box```
    * Copy AirBox KeyVisor binary and conf in to AirBox directory
    ```
    cd airbox-att-poc
    cp  kvbin/keyvisor.so /opt/air-box/
    cp  kvbin/keyvisor.conf /opt/air-box/
    ```
* There are no workflow level changes to use AirBox.
    
    * Normally, app server (tomcat) is run using the following command
    ```$(CATALINA_HOME)/bin/startup.sh```
    where $(CATALINA_HOME) is directory where tomcat server is installed, typically /opt/tomcat (see tomcat installation instructions below for details)
    
    * With AirBox, app server (tomcat) can be run in 2 steps ways:
        1. Create / edit a setenv.sh in $(CATALINA_HOME) and add following lines:
        ```
        LD_PRELOAD=/opt/air-box/keyvisor.so
        export LD_PRELOAD
        ```
        2. From $(CATALINA_HOME)/bin/ run the following command
        ```
        $ ./startup.sh
        ```
     
## Run tomcat app server with AirBox 

* You can use the following steps to run AirBox Keycentrall
    * Log in to the keycentral server
    * Clone this repo in your home directory
    ````git clone https://github.com/air-box/airbox-att-poc.git```
    * Run keycentral server
    ```
    $ cd airbox-att-poc/kcbin
    $ ./keycentral
    ```

* To test, you can use the following methods: 
    1. Command line
        ``` curl -kv https://<Tomcat IP>:8443/examples/servlets/helloworld.html```
    2. Web browser
       To use browser, you will need to add the self-signed certificates this demo uses to be trusted by the browser.
       
## Install DB server w/ AirBox (TBD)

