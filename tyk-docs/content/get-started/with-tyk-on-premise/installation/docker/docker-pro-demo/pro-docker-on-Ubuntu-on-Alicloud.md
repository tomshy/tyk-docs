---
date: 2017-03-22T16:54:02Z
title: Docker Pro Demo on Ubuntu Server 1804 on AliCloud Elasticloud
menu:
  main:
    parent: "With Docker"
weight: 2

---

> **Warning!** This demo is **NOT** designed for production use or performance testing. The Tyk Pro Docker Demo is our full, [On-Premises](https://tyk.io/api-gateway/on-premise/) solution, which includes our Gateway, Dashboard and analytics processing pipeline.
This demo will run Tyk On-Premises on your machine, which contains 5 containers: Tyk Gateway, Tyk Dashboard, Tyk Pump, Redis and MongoDB.
This demo is great for proof of concept and demo purposes, but if you want to test performance, you need to move each component to a separate machine.

### Caveat 

This guide includes broad details on setting up an Alicloud Elastic compute instance, we do not take responsibility for changes made to the the Alicloud Elastic Compute configuration pages.

Its also possible to tweak this guide to run on a free alicloud subscription licence or directly on Docker with AliCloud Container services. 

### Prerequisites

* An [AliCloud](https://account.alibabacloud.com/register/intl_register.htm) account
* A free Tyk On-Premises [Developer licence](https://tyk.io/product/tyk-on-premises-free-edition/)
* A secure shell client
* Postman for testing (Optional)

### Step One - Create a Custom Elastic Cloud Instance 

Once you have chosen your region and clicked on 'Create Instance' when you land on the basic configurations page, then:

* Choose pay as you go and Ubuntu 1804 server

* Click through to the next configuration page which is 'Networking', this is ok as Default

* Click through to the system configurations page, here choose 'password' for login credentials and create a strong password. Optionally: Change your instance name.

* Group is ok as Default

* Click through to the preview page, tick the terms and conditions box and click 'Create'

### Step Two - Connect to your instance through SSH as Root User

```$ ssh root@'your instance ip'```

When promptet, enter the password you configured when creating the instance.

### Step Three - Apply Updates and Upgrades to your Ubuntu Instance 

```$ apt-get update```
```$ apt-get upgrade```
```$ apt-get dist-upgrade```

### Step Four - Install Curl and download Docker to your Ubuntu Instance

```$ apt-get install apt-transport-https ca-certificates curl software-properties-common```

```$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -```

```$ add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"```

### Step Five - Install Docker 

```$ apt-get update```

```$ apt-get install docker-ce``` - This might take a minute or two

```$ apt-get install docker-compose```

Check that Docker is intsalled ok:

```$ systemctl status docker```

```$ docker run hello-world```

### Step Six - Install Git

```$ apt install git```

Check that Git is installed:

```$ git --version```

### Step Six - Download and Install Tyk!

```$ git clone https://github.com/TykTechnologies/tyk-pro-docker-demo.git```

Add your free Tyk Developer licence key to the analytics.conf file:

```$ nano tyk-pro-docker-demo/confs/tyk_analytics.conf```

Enter the licence key into the ```Licence_key=""``` variable 

Exit the editor

### Step Seven - Launch Tyk 

```$ cd tyk-pro-docker-demo```

```$ docker-compose -f docker-compose.yml -f docker-local.yml  up```

### Step Eight - Run the Set Up Script

Login into the instance through a new secure shell, following the same steps as before:

```$ ssh root@'your instance ip'```

When promptet, enter the password you configured when creating the instance.

Once you have accessed the instance through a new secure shell, then:

```$ cd tyk-pro-docker-demo```

```$ chmod +x setup.sh```

```$ ./setup.sh```

Take note of the User name and Password provided by the Set up script. 

Ignore the login address, this will be based on your instance ip.

### Step Eight - Open Ports on your AliCloud Instance

Note: this is not secure but a general idea on how to open ports on an alicloud instance:
In the Alicloud console, click on your instance and choose Security Groups, click add rule and then add security group rule, in port range add 3000, In authorization objects, add 0.0.0.0/0

### Step 9 - Access the Dashboard

In your local browser, Navigate to “your instance ip”:3000

### Step 10 -

That's it, you've set up all of the Tyk Pro components on Docker on Ubuntu on a single AliCloud instance. 

Don't forget to shutdown the instance before incurring excess charges.



