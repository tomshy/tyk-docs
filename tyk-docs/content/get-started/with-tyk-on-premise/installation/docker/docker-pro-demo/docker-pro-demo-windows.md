---
date: 2017-03-22T16:54:02Z
title: Docker Pro Demo on Windows
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

* MS Windows 10 Pro
* [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/) running with a signed in [Docker ID](https://docs.docker.com/docker-id/)
* Git for Windows
* PowerShell running as administrator
* Postman for [Windows](https://www.getpostman.com/downloads/)
* Our Pro Demo Docker [GitHub repo](https://github.com/TykTechnologies/tyk-pro-docker-demo)
* A free Tyk On-Premises [Developer licence](https://tyk.io/product/tyk-on-premises-free-edition/)

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

### Step Five - Test the Tyk Dashboard URL

Go to:

```{copy.Wrapper}
127.0.0.1:3000
```

You should get to the Tyk Dashboard login screen:

![Tyk Dashboard Login Screen][1]

### Step Six - Create an Organisation via Postman

You need to create an organisation for your initial user to belong to.

* **Request**: POST
* **URL**: `127.0.0.1:3000/admin/organisations/`
* **Header**: Key `admin-auth` Value `12345`
* **Body** (raw set to application/json):

#### Sample Request

```{copy.Json}
{"owner_name": "TestOrg5 Ltd.",
"owner_slug": "testorg",
"cname_enabled":true,
"cname": "localhost"}
```

This will return a `"org_id"` value. You need this value to create your user.

#### Sample Response

```
{
    "Status": "OK",
    "Message": "Org created",
    "Meta": "5d07b4b0661ea80001b3d40d"
}
```


### Step Seven - Create a User via Postman

This sets up an initial user to enable you to login to your Dashboard.

* **Request**: POST
* **URL**: `127.0.0.1:3000/admin/users`
* **Header**: Key `admin-auth` Value `12345`
* **Body** (raw set to application/json):

#### Sample Request

```{copy.Json}
{"first_name": "",
"last_name": "",
"email_address": "",
"active": true,
"org_id": "5d07b4b0661ea80001b3d40d"}
```

Enter a `first_name`, `last_name`, `email_address` and the `org_id` value created in **Step 6**.

This creates a `access_key` value. You need this to create a password for the user.

#### Sample Response

```
{
    "Status": "OK",
    "Message": "e1d655236f4e43cf687d83ac3b2a5054",
    "Meta": {
        "api_model": {},
        "first_name": "Mark",
        "last_name": "Southee",
        "email_address": "mark+8@tyk.io",
        "org_id": "5d07b4b0661ea80001b3d40d",
        "active": true,
        "id": "5d07b4b0661ea80001b3d40d",
        "access_key": "e1d655236f4e43cf687d83ac3b2a5054",
        "user_permissions": {
            "IsAdmin": "admin",
            "ResetPassword": "admin"
        },
        "group_id": "",
        "password_max_days": 0,
        "password_updated": "0001-01-01T00:00:00Z",
        "PWHistory": [],
        "created_at": "2019-06-20T09:43:07.839Z"
    }
}
```



### Step Eight - Add a password for your User

This creates a password for the user setup in **Step Seven**. Replace `USER_ID` with your `user_id` value. For the `Authorization` Header, the Value you need to enter is the `access_key` value from the create user request. 

* **Request**: POST
* **URL**: `127.0.0.1:3000/api/users/USER_ID/actions/reset`
* **Header**: Key `Authorzation` Value `SECRET_VALUE`
* **Body** (raw set to application/json):

#### Sample Request

```{copy.Json}
{"new_password":"Password"}
```

#### Sample Response

```
{
    "Status": "OK",
    "Message": "User password updated",
    "Meta": null
}
```


### Step Nine - Set up a Portal Catalogue

This creates a portal catalogue for your developer portal. For the `Authorization` Header, the Value you need to enter is the `access_key` value from the create user request. In the body add the `org_id` value created in **Step One**.

* **Request**: POST
* **URL**: `127.0.0.1:3000/api/portal/catalogue`
* **Header**: Key `Authorzation` Value `SECRET_VALUE`
* **Body** (raw set to application/json):

#### Sample Request

```{copy.Json}
{"org_id": "5d07b4b0661ea80001b3d40d"}
```

#### Sample Response

```
{
    "Status": "OK",
    "Message": "5d07b4b0661ea80001b3d40d",
    "Meta": null
}
```

### Step Ten - Create your default Portal Pages

This creates the default home page for your developer portal. For the `Authorization` Header, the Value you need to enter is the `access_key` value from the create user request.

* **Request**: POST
* **URL**: `127.0.0.1:3000/api/portal/catalogue`
* **Header**: Key `Authorzation` Value `SECRET_VALUE`
* **Body** (raw set to application/json):

#### Sample Request

```{copy.Json}
{"is_homepage": true,
"template_name":"", "title":"Tyk Developer Portal",
"slug":"home",
"fields":
{"JumboCTATitle": "Tyk Developer Portal",
"SubHeading": "Sub Header",
"JumboCTALink": "#cta",
"JumboCTALinkTitle": "Your awesome APIs, hosted with Tyk!",
"PanelOneContent": "Panel 1 content.",
"PanelOneLink": "#panel1",
"PanelOneLinkTitle": "Panel 1 Button",
"PanelOneTitle": "Panel 1 Title",
"PanelThereeContent": "",
"PanelThreeContent": "Panel 3 content.",
"PanelThreeLink": "#panel3",
"PanelThreeLinkTitle": "Panel 3 Button",
"PanelThreeTitle": "Panel 3 Title",
"PanelTwoContent": "Panel 2 content.",
"PanelTwoLink": "#panel2",
"PanelTwoLinkTitle": "Panel 2 Button",
"PanelTwoTitle": "Panel 2 Title"}
}
```

#### Sample Response

```
{
    "Status": "OK",
    "Message": "5d07b4b0661ea80001b3d40d",
    "Meta": null
}
```

### Step Eleven - Setup the Portal URL

This creates the developer portal URL. For the `Authorization` Header, the Value you need to enter is the `secret` value from your `/confs/tyk_analytics.conf`.

* **Request**: POST
* **URL**: `127.0.0.1:3000/api/portal/configuration`
* **Header**: Key `Authorzation` Value `SECRET_VALUE`
* **Body** (raw set to application/json):

#### Sample Request

```{copy.Json}
{}
```


#### Sample Response

```
{
    "Status": "OK",
    "Message": "5d07b4b0661ea80001b3d40d",
    "Meta": null
}
```



[1]: /docs/img/dashboard/system-management/dashboard_login.png
