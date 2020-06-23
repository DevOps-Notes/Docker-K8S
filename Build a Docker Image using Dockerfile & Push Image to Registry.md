# Build a Docker Image using Dockerfile & Push Image to Registry

* Install Maven

* Upload the SpringBoot project (whole folder) to server, then build it with Maven

  ```sh
  scp AzureMonitoringSuite_2020-05-25_2313.zip root@master2:/root/
  unzip -q AzureMonitoringSuite_2020-05-25_2313.zip
  cd AzureMonitoringSuite/
  mvn clean install
  mv /root/.m2/repository/com/ams/main/0.0.1-SNAPSHOT/main-0.0.1-SNAPSHOT.war /root/ROOT.war
  ```

* vim Dockerfile

  ```sh
  FROM tomcat
  RUN rm -rf /usr/local/tomcat/webapps/*
  COPY ROOT.war /usr/local/tomcat/webapps/
  ```

* Install Docker & Build Image

  ```sh
  yum -y install docker
  systemctl daemon-reload
  service docker restart
  service docker status
  docker build -t tomcat:ams .
  docker image ls
  ```

* Install Azure CLI using yum

  ```sh
  rpm --import https://packages.microsoft.com/keys/microsoft.asc
  
  sh -c 'echo -e "[azure-cli]
  name=Azure CLI
  baseurl=https://packages.microsoft.com/yumrepos/azure-cli
  enabled=1
  gpgcheck=1
  gpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/azure-cli.repo'
  
  yum -y install azure-cli
  az cloud set --name AzureChinaCloud
  az login
  az configure --defaults acr=wluocr
  az acr login
  ```

* Push Image to Azure Container Registry

  ```sh
  docker tag tomcat:ams wluocr.azurecr.cn/tomcat:ams
  docker push wluocr.azurecr.cn/tomcat:ams
  ```

* Pull Image from Azure Container Registry

  ```sh
  # Remove the local image first
  docker rmi wluocr.azurecr.cn/tomcat:ams
  docker run -d -p 80:8080 wluocr.azurecr.cn/tomcat:ams
  ```

  

