Documentation of openHAB's security features

## Introduction

To secure the communication with openHAB there are currently two mechanisms in place

- HTTPS
- Authentication

Authentication is implemented by `SecureHttpContext` which in turn implements `HttpContext`. This `SecureHttpContext` is registered with the OSGi !HttpService and provides the security hook `handleSecurity`. At least all authentication requests are delegated to the `javax.security.auth.login.LoginContext.LoginContext` which is the entry point to JAAS (http://en.wikipedia.org/wiki/Java_Authentication_and_Authorization_Service) !LoginModules.

The `SecureHttpContext` is currently used by the `WebAppServlet` and the `CmdServlet` which constitutes the default iPhone UI as well as the `RESTApplication` which provides the REST functionality.

## HTTPS

openHAB supports HTTPS out of the box. Just point your browser to 

`https://127.0.0.1:8443/openhab.app?sitemap=demo#` 

and the HTTP communication will be encrypted by SSL. 

If you prefer to use your own X.509 certificates, you can. [Configure_SSL](http://wiki.eclipse.org/Jetty/Howto/Configure_SSL) has information on how to do that, and there's a
[step-by-step guide](https://gist.github.com/jpmens/8029383) specifically for openHAB users.

[Here is a guide] (https://gist.github.com/dfraser/e5710fb84c6d35aa0743) for using [Let's Encrypt] (https://letsencrypt.org) with openHAB.

## Authentication

In order to activate Authentication one has to have the following parameters to the openHAB start command line.  This may already be done by default.  

- `-Djava.security.auth.login.config=./etc/login.conf` - the configuration file of the JAAS !LoginModules

By default the command line references the file `<openhabhome>/etc/login.conf` which in turn configures a PropertyFileLoginModule that references the user configuration file `login.properties`. One should use all available LoginModule implementation here as well (see http://wiki.eclipse.org/Jetty/Tutorial/JAAS for further information).

The default configuration for login credentials for openHAB is the file  `<openhabhome>/configuration/users.cfg`. In this file, you can put a simple list of "user=pwd" pairs, which will then be used for the authentication.  For example:

```
user=password,user,role
joe=password1
wendy=12345
```

Note that you could optionally add roles after a comma, but there is currently no support for different roles in openHAB.

## Security Options

The security options can be configured through `openhab.cfg`. One can choose between

- `ON` - security is enabled generally
- `OFF` - security is disabled generally
- `EXTERNAL` - security is switched on for external requests (e.g. originating from the Internet) only

To distinguish between internal and external addresses one may configure the `security:netmask` in `openhab.cfg`. Every ip-address which is in range of this net mask will be treated as internal address must not be authorized though. Multiple netmasks can be specified (comma separated). This may be necessary on networks using both IPv4 and IPv6 addresses. To debug issues with netmasks set the `org.openhab.io.net.http` logging level to TRACE in logback.xml.