# KN06
[Task Description](./task/KN06.pdf)

## Steps
---

## A)

### 1.

To start with this exercise I will be able to reuse the network interfaces of [KN05](../KN05/KN05_doc.md).

I will also use the same ip addresses for my database and web server I used for [KN05](../KN05/KN05_doc.md):
- Database private IPv4: `172.31.10.10`
- Webserver private IPv4: `172.31.10.20`

First of all I will setup a new EC2 database Instance with the following cloud-init configuration:

```yaml
#cloud-config
users:
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    home: /home/ubuntu
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC0WGP1EZykEtv5YGC9nMiPFW3U3DmZNzKFO5nEu6uozEHh4jLZzPNHSrfFTuQ2GnRDSt+XbOtTLdcj26+iPNiFoFha42aCIzYjt6V8Z+SQ9pzF4jPPzxwXfDdkEWylgoNnZ+4MG1lNFqa8aO7F62tX0Yj5khjC0Bs7Mb2cHLx1XZaxJV6qSaulDuBbLYe8QUZXkMc7wmob3PM0kflfolR3LE7LResIHWa4j4FL6r5cQmFlDU2BDPpKMFMGUfRSFiUtaWBNXFOWHQBC2+uKmuMPYP4vJC9sBgqMvPN/X2KyemqdMvdKXnCfrzadHuSSJYEzD64Cve5Zl9yVvY4AqyBD aws-key  
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6lpo3pXfnBCXISVAapHYlDd5uybJlQrcGstabUbHAWMoSRMGMTSrMx4YP4wnUR2Zrv68n8Nm7ZonyV77CMQP7jVVlMwhw0bA9TEsCvEOmo9KQPwn6WoH2DequDsCTZtHrFFfZrs+V9CuUP28GQPaZyMprB4cP7a906BHLfx6C1spHWF4CcVuNM7lJGFaN+roX6XKu9uhrwY0LovifpZu83rmANNh76pXgWEBnfNzV5FM8k98z2u9enbAHtRwuc/r0qoHurAoMhiL8RlfeqhgCFXq5H+8xKIu/M/tvxBXYGwZw+7a6rnPkKFdVBDM5kf44d9fDsaj94stftFfi8Ls9 aws-key  
ssh_pwauth: true
disable_root: false  
packages:
  - mariadb-server
runcmd:
  - sudo mysql -sfu root -e "GRANT ALL ON *.* TO 'admin'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;"
  - sudo sed -i 's/127.0.0.1/0.0.0.0/g' /etc/mysql/mariadb.conf.d/50-server.cnf
  - sudo systemctl restart mariadb.service
  - sudo cd /home/ubuntu
  - sudo git clone https://gitlab.com/ch-tbz-it/Stud/m346/m346scripts.git
  - sudo mysql -sfu root < ./m346scripts/KN06/shop-database.sql
```

For the operating system of the instance I choose `Ubuntu` and for the instance type `t1.micro`. The network interface I choose is called `KN05-database-network-interface`.
The storage size I can leave untouched at `1x 8GB`.

### 2.

Next up lets install the web server. Here I have two options, either use a .NET or Java Application. Because I have more experience with Java I will use the Java Application. 

Following is the cloud-init configuration of the web server instance:

```yaml
#cloud-config
users:
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    home: /home/ubuntu
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC0WGP1EZykEtv5YGC9nMiPFW3U3DmZNzKFO5nEu6uozEHh4jLZzPNHSrfFTuQ2GnRDSt+XbOtTLdcj26+iPNiFoFha42aCIzYjt6V8Z+SQ9pzF4jPPzxwXfDdkEWylgoNnZ+4MG1lNFqa8aO7F62tX0Yj5khjC0Bs7Mb2cHLx1XZaxJV6qSaulDuBbLYe8QUZXkMc7wmob3PM0kflfolR3LE7LResIHWa4j4FL6r5cQmFlDU2BDPpKMFMGUfRSFiUtaWBNXFOWHQBC2+uKmuMPYP4vJC9sBgqMvPN/X2KyemqdMvdKXnCfrzadHuSSJYEzD64Cve5Zl9yVvY4AqyBD aws-key  
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6lpo3pXfnBCXISVAapHYlDd5uybJlQrcGstabUbHAWMoSRMGMTSrMx4YP4wnUR2Zrv68n8Nm7ZonyV77CMQP7jVVlMwhw0bA9TEsCvEOmo9KQPwn6WoH2DequDsCTZtHrFFfZrs+V9CuUP28GQPaZyMprB4cP7a906BHLfx6C1spHWF4CcVuNM7lJGFaN+roX6XKu9uhrwY0LovifpZu83rmANNh76pXgWEBnfNzV5FM8k98z2u9enbAHtRwuc/r0qoHurAoMhiL8RlfeqhgCFXq5H+8xKIu/M/tvxBXYGwZw+7a6rnPkKFdVBDM5kf44d9fDsaj94stftFfi8Ls9 aws-key  
ssh_pwauth: true
disable_root: false  
package_update: true
packages:
  - curl
  - wget
  - nginx
  - openjdk-18-jre
write_files:
  - content: |
      [Unit]
      Description=Demo Shop
      After=syslog.target
      [Service]
      WorkingDirectory=/var/www/app
      User=www-data
      ExecStart=java -jar /var/www/app/shopdemo-1.0.0.jar --spring.config.additional-location=production.properties SuccessExitStatus=143 
      Restart=always
      RestartSec=10
      [Install] 
      WantedBy=multi-user.target
    path: /etc/systemd/system/shop-app.service
  - content: |
      server {
        listen        80;
        server_name   50.17.215.47;
        location / {
          proxy_pass         http://127.0.0.1:5001;
          proxy_http_version 1.1;
          proxy_set_header   Upgrade $http_upgrade;
          proxy_set_header   Connection keep-alive;
          proxy_set_header   Host $host;
          proxy_cache_bypass $http_upgrade;
          proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header   X-Forwarded-Proto $scheme;
        }
      }
    path: /home/ubuntu/nginx
runcmd:
  - sudo mkdir /var/www
  - sudo mkdir /var/www/app
  - sudo git clone https://gitlab.com/ch-tbz-it/Stud/m346/m346scripts.git
  - sudo sed -i 's/<your-connection-string>/jdbc:mysql:\/\/172.31.10.10:3306\/shop/g' ./m346scripts/KN06/java/production.properties
  - sudo cp -r ./m346scripts/KN06/java/* /var/www/app/
  - sudo systemctl enable shop-app.service
  - sudo systemctl start shop-app.service
  - sudo service nginx start
  - sudo cp /home/ubuntu/nginx /etc/nginx/sites-available/default
  - sudo nginx -s reload
```

I have to addapt some placeholders in the could-init template like the private ip address of the database instance (`172.31.10.10`) and the public ip address of the web server instance (Elastic IP: `50.17.215.47`).

To launch the instance I also addapt some settings, OS: `Ubuntu`, interface type `t1.micro` with `1x 8GB` storage and select the network interface of my web server (`KN05-webserver-network-interface`).

To check if my web application is working I call the following Swagger url and check out the response: `http://50.17.215.47/swagger-ui/index.html`.

![aws_instance_web_server_swagger.png](./images/aws_instance_web_server_swagger.PNG)

As we can see in the screenshot above, Swagger works and I am able to check out the applications endpoints. 

Next up I make sure the database connection is working as well. Therefor I test one of the endpoints (in my case `/shop/alive`). Via the Swagger UI I can try the request.

![aws_instance_web_server_shop_alive.png](./images/aws_instance_web_server_shop_alive.PNG)

The server responds with "I'm alive" which is a good sign. 

## B)

For the next exercise I will scale my web server and database instances vertically by increasing the disk size of the database instance to `20GB` and the instance type of the web server instance to `t2.medium`. The whole scaling will be made during ongoing operations.

Below are screenshots of the resource settings of both of my instances **before** the scaling:

`KN06-Database`

![aws_instance_database_disk_size_before.png](./images/aws_instance_database_disk_size_before.PNG)

`KN06-Webserver`

![aws_instance_web_server_type_before.png](./images/aws_instance_web_server_type_before.PNG)

To change the instance type I first have to stop the web server instance and then select `Actions -> Instance settings -> Change instance type`.

To increase the disk size I have to stop the database instance, navigate to the `Volumes` tab and on my database instance volume select `Actions -> Modify volume`.

Now I can start both of my instances again. Below are screenshots of the resource settings of both of my instances **after** the scaling:

`KN06-Database`

![aws_instance_database_disk_size_after.png](./images/aws_instance_database_disk_size_after.PNG)

`KN06-Webserver`

![aws_instance_web_server_type_after.png](./images/aws_instance_web_server_type_after.PNG)

## C)

In the following exercise I will create a Load Balancer for horizontal scaling of my instances. 

I navigate to the section `Load Balancers` and create a new `Application Load Balancer`.

- Start second instance of web server manually


Neue Instance von Webserver, Neue Network interface von Webserver mit zweiter elastic ip

Public ip von database entfernen
SSH Inbound von DB entfernen