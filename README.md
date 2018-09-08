# My Code Explanation

Resources:

### Resources for the EC2 Instance

  EC2app:
Type: AWS::EC2::Instance

Properties: 

InstanceType: t2.micro (the smallest and cheapest)

### i chose ubuntu 16.04 in us-west-2)

ImageId: 

### key pair that i generated

keyname :aws-key1 


Securitygroups:
-!Ref ec2Appsecgroup

### source list for docker source and install docker
### standard set of commands that would run on Ubuntu to install docker

UserData: !Base64 |
        #!/bin/bash
        apt-get update -qq
        apt-get install -y apt-transport-https ca-certificates
        apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
        echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | tee /etc/apt/sources.list.d/docker.list
        apt-get update -qq
        apt-get purge lxc-docker || true
        apt-get -y install linux-image-extra-$(uname -r) linux-image-extra-virtual
        apt-get -y install docker-engine
        usermod -aG docker ubuntu
        mkdir -p /etc/systemd/system/docker.service.d
        printf "[Service]\nExecStart=\nExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375\n" >>  /etc/systemd/system/docker.service.d/docker.conf
        systemctl daemon-reload
        systemctl restart docker

### Resources for security group to allow inbound ports

ec2Appsecgroup:
Type: AWS::EC2::SecurityGroup
Properties:
GroupDescription: for my app to allow ssh, http and docker ports

### defined an array or a list of ingress or inbound connection properties

SecurityGroupIngress: 
-IpProtocol:tcp (Http port)
FromPort: '80' 
ToPort: '80'
CidrIp: 0.0.0.0/0 (it will be reachable anywhere)
-IpProtocol:tcp (SSH port)
FromPort: '22' 
ToPort: '22'
CidrIp: 0.0.0.0/0 (it will be reachable anywhere)
- IpProtocol: tcp (Docker port) 
        FromPort: '2375'
        ToPort: '2375'
        CidrIp: 0.0.0.0/0

### Resources for DB Instance
### I used RDS for MySQL

 DatabaseInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBName: "blog"
      Engine: MySQL
      MasterUsername: bloguser
      MasterUserPassword: password123
      DBInstanceClass: db.t2.micro
      AllocatedStorage: '5'
      DBSecurityGroups:
        - !Ref DatabaseSG

### Resources for DB Security Group

  DatabaseSG:
    Type: AWS::RDS::DBSecurityGroup
    Properties:
      GroupDescription: Security Group for RDS public access
      DBSecurityGroupIngress:
        - CIDRIP: 0.0.0.0/0

### Docker Compose file

version: '2'
### defined 1 service name as app
services:
  app:
    image: devteds/rails-api-example
    ports:
      - "90:2000"
    environment:
      DB_USER: bloguser
      DB_NAME: blog
      DB_PASSWORD: password123
      DB_HOST: bd1p90upedqrzuv.cyj36qazkf1j.us-west-2.rds.amazonaws.com
      RAILS_ENV: production
      RAILS_LOG_TO_STDOUT: 1
      SECRET_KEY_BASE: 5156ca5a10c6842c4359d9a78ba14a4ba91cee9394d2e87bb783a0f770469d6afd725d40346d328b705cb2b74ef7de2005701c3449a1dbd1d1bc163ce3b67ec1
