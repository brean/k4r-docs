# Identity and Access Management
## Introduction
Knowledge4Retail uses Oauth2 and OpenID Connect and  for Identity and Access Management. By default, the services rely [!external OAUTH authentication](https://kubernetes.github.io/ingress-nginx/examples/auth/oauth-external-auth/) at the Ingress-stage. Depending on the use-case, authorization decisions can be made on the Ingress-level or directly within the upstream service using the provided OIDC identity information. It is possible to connect the reverse-proxy authentication directly to your corporate identity provider or by chaining with the [!provided keycloak installation](https://github.com/knowledge4retail/k4r-infrastructure). For simple POC-like scenarios in protected demo environments with no authoritation requirements, X.509 based authentication may be used.
## Technical Implementation
The provided K4R [!reference installation](https://github.com/knowledge4retail/k4r-infrastructure) uses [!oauth2-proxy](https://github.com/oauth2-proxy/oauth2-proxy) for authentication at the Ingress-stage. Oauth2-proxy is configured to authenticate with an integrated keycloak and, given a positive authorization decision, produce access and identity token which are then provided to the upstream k4r services. Services requiring role-based access control can be configured accordingly by amending their ingress configuration. Please refer to the oauth2-proxy documentation on how to accomplish this. Alternatively, object-based authorization decisions should be implemented within the upstream services.\
![Dropdown](resources/oauth2.png)

If X.509 authentication should be used for demonstration purposes, please refer to the samples within the [!infrastructure repository](https://github.com/knowledge4retail/k4r-infrastructure)

## Customization
If you want to provide your own IDP, consider if it is supported by oauth2-proxy directly and exposed to the internet. If so, you can connect oauth2-proxy directly. If your IDP is internal or not supported, you can configure the provided keycloak for identity brokering with your IDP. In this case, you also need to create a role mapping to match your configured oauth2-proxy access roles.

## Samples

### Machine to machine
The following request shows how to authenticate with keycloak and acquire an OIDC token set by using the Password-Credentials flow:\
![Dropdown](resources/pw.png)
Plain HTTP:
```code
POST /auth/realms/k4r/protocol/openid-connect/token HTTP/1.1
Host: auth.sandbox.knowledge4retail.org
Content-Type: application/x-www-form-urlencoded
Content-Length: 119
 
username=k4rtest&password=redacted&client_id=k4r-test&grant_type=password
```

Consuming a protected k4r service

The following request shows how to use the acquired access token in the authorization header:
![Dropdown](resources/pw2.png)
Plain HTTP:
```code
GET /k4r/api/v0/stores/1026 HTTP/1.1
Host: dt-api-oidc.sandbox.knowledge4retail.org
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJMZUlMSjhmUVd[...]
```

### Interactive
Please refer to the sample configuration of the central user interface for an end-to-end example of how to protect an interactive k4r service. The application uses an angular frontend and a kotlin backend to demonstrate how to implement auth-code based authentication.