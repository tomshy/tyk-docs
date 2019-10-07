---
date: 2017-03-22T16:54:02Z
title: Docker Pro Demo on Windows Linux Subsystem
menu:
  main:
    parent: "With Docker"
weight: 2

---

> **Warning!** This demo is **NOT** designed for production use or performance testing. The Tyk Pro Docker Demo is our full, [On-Premises](https://tyk.io/api-gateway/on-premise/) solution, which includes our Gateway, Dashboard and analytics processing pipeline.
This demo will run Tyk On-Premises on your machine, which contains 5 containers: Tyk Gateway, Tyk Dashboard, Tyk Pump, Redis and MongoDB.
This demo is great for proof of concept and demo purposes, but if you want to test performance, you need to move each component to a separate machine.

> **NOTE**: You use this at your own risk. Tyk is not supported on the Windows platform. However you can test it as a proof of concept using our Pro Demo Docker installation.


## Prerequisites

* MS Windows 10 Pro with [Windows Linux Subsystem](https://docs.microsoft.com/en-us/windows/wsl/install-win10) enabled
* [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/) running with a signed in [Docker ID](https://docs.docker.com/docker-id/)
* Git for Windows
* PowerShell running as administrator
* Postman for [Windows](https://www.getpostman.com/downloads/)
* Our Pro Demo Docker [GitHub repo](https://github.com/TykTechnologies/tyk-pro-docker-demo)
* A free Tyk On-Premises [Developer licence](https://tyk.io/product/tyk-on-premises-free-edition/)
* Optional: Ubuntu on Windows

### Step One - Clone the Repo

Clone the repo above to a location on your machine.

### Step Two - Edit your hosts file

You need to add the following to your Windows hosts file:

```{copy.Wrapper}
127.0.0.1 www.tyk-portal-test.com
127.0.0.1 www.tyk-test.com
```

### Step Three - Add your Developer Licence

You should have received your free developer licence via email. Copy the licence key in the following location from your `\confs\tyk_analytics.conf` file:

```
"license_key": ""
```

### Step Four - Run the Docker Compose File

From PowerShell, run the following command from your installation folder:

```{copy.Wrapper}
docker-compose -f docker-compose.yml -f docker-local.yml up
```

This will will download and setup the five Docker containers. This may take some time and will display all output.

### Step Five - Run the Setup Script

Open another Powershell Terminal in Administrator Mode from the same folder as the setup script, run the command:
```{copy.Wrapper}
bash
```

This will open a bash prompt in Powershell, now run:
```{copy.Wrapper}
./setup.sh
```

### Alternatively - Run the Setup Script

Open a WSL terminal in the same folder as the setup script and run the script.  

The Setup script will run for a short time and provide you with login details, including login IP, along with a sample User Name and Password. 

Follow the login details and view the Tyk Dashboard. ....


