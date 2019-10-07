---
date: 2019-08-09T12:47:13+01:00
title: Dashboard API Single Sign On
menu:
  main:
    parent: "Tyk Dashboard API"
weight: 12
---

The Dashboard SSO API allows you to implement custom authentication schemes for the Dashboard and Portal. 
Our Tyk Identity Broker (TIB) internally also uses this API.

### Generate authentication token

The Dashboard exposes the `/api/sso` Dashboard API which allows you to generate a temporary authentication token, valid for 60 seconds. 

You should provide JSON payload with the following data:

* `ForSection` - scope with possible values of `"dashboard"` or `"portal"` 
* `OrgID` - with your organisation id. 
* `GroupID` - the group id 
* `EmailAddress` - user email 


| **Property** | **Description**              |
| ------------ | ---------------------------- |
| Resource URL | `/api/sso` |
| Method       | POST                         |
| Body         | `{"ForSection":"<scope>", "OrgID": "<org-id>", "GroupID": "<group-id>"}`  |

#### Sample Request

```{.copyWrapper}
POST /api/sso HTTP/1.1
Host: localhost:3000
authorization:7a7b140f-2480-4d5a-4e78-24049e3ba7f8
    
{
  "ForSection": "dashboard",
  "OrgID": "588b4f0bb275ff0001cc7471",
  "EmailAddress": "name@somewhere.com",
  "GroupID": ""
}
```

#### Sample Response:
```{.copyWrapper}
{"Status":"OK","Message":"SSO Nonce created","Meta":"YTNiOGUzZjctYWZkYi00OTNhLTYwODItZTAzMDI3MjM0OTEw"}
```

See [Single Sign On](/docs/integrate/sso) documentation for how to use this token and more details.
