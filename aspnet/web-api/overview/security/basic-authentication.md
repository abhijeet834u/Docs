---
title: "Basic Authentication in ASP.NET Web API | Microsoft Docs"
author: MikeWasson
description: "Describes using Basic Authentication in ASP.NET Web API."
ms.author: riande
manager: wpickett
ms.date: 10/02/2014
ms.topic: article
ms.assetid: 
ms.technology: dotnet-webapi
ms.prod: .net-framework
msc.legacyurl: /web-api/overview/security/basic-authentication
---
[Edit .md file](C:\Projects\msc\dev\Msc.Www\Web.ASP\App_Data\github\web-api\overview\security\basic-authentication.md) | [Edit dev content](http://www.aspdev.net/umbraco#/content/content/edit/43543) | [View dev content](http://docs.aspdev.net/tutorials/web-api/overview/security/basic-authentication.html) | [View prod content](http://www.asp.net/web-api/overview/security/basic-authentication) | Picker: 43551

Basic Authentication in ASP.NET Web API
====================
by [Mike Wasson](https://github.com/MikeWasson)

Basic authentication is defined in [RFC 2617, HTTP Authentication: Basic and Digest Access Authentication](http://www.ietf.org/rfc/rfc2617.txt).

| Advantages | Disadvantages |
| --- | --- |
| - Internet standard. - Supported by all major browsers. - Relatively simple protocol. | - User credentials are sent in the request. - Credentials are sent as plaintext. - Credentials are sent with every request. - No way to log out, except by ending the browser session. - Vulnerable to cross-site request forgery (CSRF); requires anti-CSRF measures. |

Basic authentication works as follows:

1. If a request requires authentication, the server returns 401 (Unauthorized). The response includes a WWW-Authenticate header, indicating the server supports Basic authentication.
2. The client sends another request, with the client credentials in the Authorization header. The credentials are formatted as the string "name:password", base64-encoded. The credentials are not encrypted.

Basic authentication is performed within the context of a "realm." The server includes the name of the realm in the WWW-Authenticate header. The user's credentials are valid within that realm. The exact scope of a realm is defined by the server. For example, you might define several realms in order to partition resources.

![](basic-authentication/_static/image1.png)

Because the credentials are sent unencrypted, Basic authentication is only secure over HTTPS. See [Working with SSL in Web API](working-with-ssl-in-web-api.md).

Basic authentication is also vulnerable to CSRF attacks. After the user enters credentials, the browser automatically sends them on subsequent requests to the same domain, for the duration of the session. This includes AJAX requests. See [Preventing Cross-Site Request Forgery (CSRF) Attacks](preventing-cross-site-request-forgery-csrf-attacks.md).

## Basic Authentication with IIS

IIS supports Basic authentication, but there is a caveat: The user is authenticated against their Windows credentials. That means the user must have an account on the server's domain. For a public-facing web site, you typically want to authenticate against an ASP.NET membership provider.

To enable Basic authentication using IIS, set the authentication mode to "Windows" in the Web.config of your ASP.NET project:

    <system.web>
        <authentication mode="Windows" />
    </system.web>

In this mode, IIS uses Windows credentials to authenticate. In addition, you must enable Basic authentication in IIS. In IIS Manager, go to Features View, select Authentication, and enable Basic authentication.

![](basic-authentication/_static/image2.png)

In your Web API project, add the `[Authorize]` attribute for any controller actions that need authentication.

A client authenticates itself by setting the Authorization header in the request. Browser clients perform this step automatically. Nonbrowser clients will need to set the header.

## Basic Authentication with Custom Membership

As mentioned, the Basic Authentication built into IIS uses Windows credentials. That means you need to create accounts for your users on the hosting server. But for an internet application, user accounts are typically stored in an external database.

The following code how an HTTP module that performs Basic Authentication. You can easily plug in an ASP.NET membership provider by replacing the `CheckPassword` method, which is a dummy method in this example.

In Web API 2, you should consider writing an [authentication filter](authentication-filters.md) or [OWIN middleware](../../../aspnet/overview/owin-and-katana/index.md), instead of an HTTP module.

    namespace WebHostBasicAuth.Modules
    {
        public class BasicAuthHttpModule : IHttpModule
        {
            private const string Realm = "My Realm";
    
            public void Init(HttpApplication context)
            {
                // Register event handlers
                context.AuthenticateRequest += OnApplicationAuthenticateRequest;
                context.EndRequest += OnApplicationEndRequest;
            }
    
            private static void SetPrincipal(IPrincipal principal)
            {
                Thread.CurrentPrincipal = principal;
                if (HttpContext.Current != null)
                {
                    HttpContext.Current.User = principal;
                }
            }
    
            // TODO: Here is where you would validate the username and password.
            private static bool CheckPassword(string username, string password)
            {
                return username == "user" && password == "password";
            }
    
            private static void AuthenticateUser(string credentials)
            {
                try
                {
                    var encoding = Encoding.GetEncoding("iso-8859-1");
                    credentials = encoding.GetString(Convert.FromBase64String(credentials));
    
                    int separator = credentials.IndexOf(':');
                    string name = credentials.Substring(0, separator);
                    string password = credentials.Substring(separator + 1);
    
                    if (CheckPassword(name, password))
                    {
                        var identity = new GenericIdentity(name);
                        SetPrincipal(new GenericPrincipal(identity, null));
                    }
                    else
                    {
                        // Invalid username or password.
                        HttpContext.Current.Response.StatusCode = 401;
                    }
                }
                catch (FormatException)
                {
                    // Credentials were not formatted correctly.
                    HttpContext.Current.Response.StatusCode = 401;
                }
            }
    
            private static void OnApplicationAuthenticateRequest(object sender, EventArgs e)
            {
                var request = HttpContext.Current.Request;
                var authHeader = request.Headers["Authorization"];
                if (authHeader != null)
                {
                    var authHeaderVal = AuthenticationHeaderValue.Parse(authHeader);
    
                    // RFC 2617 sec 1.2, "scheme" name is case-insensitive
                    if (authHeaderVal.Scheme.Equals("basic",
                            StringComparison.OrdinalIgnoreCase) &&
                        authHeaderVal.Parameter != null)
                    {
                        AuthenticateUser(authHeaderVal.Parameter);
                    }
                }
            }
    
            // If the request was unauthorized, add the WWW-Authenticate header 
            // to the response.
            private static void OnApplicationEndRequest(object sender, EventArgs e)
            {
                var response = HttpContext.Current.Response;
                if (response.StatusCode == 401)
                {
                    response.Headers.Add("WWW-Authenticate",
                        string.Format("Basic realm=\"{0}\"", Realm));
                }
            }
    
            public void Dispose() 
            {
            }
        }
    }

To enable the HTTP module, add the following to your web.config file in the **system.webServer** section:

[!code[Main](basic-authentication/samples/sample1.xml?highlight=4)]

Replace "YourAssemblyName" with the name of the assembly (not including the "dll" extension).

You should disable other authentication schemes, such as Forms or Windows auth.