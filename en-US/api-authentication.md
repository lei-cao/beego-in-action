The API Authentication
=========

Providing API authentication is a challenge work.

The HTTP protocol is *stateless* which means each HTTP request to the server is independent transaction and unrelated to any previous request.

But in many cases we want to keep the session in order to communicate between the client and the server in a set of requests. So what's the solution?

#Cookie-Based Authentication

The traditional way is using **Cookie-Based Authentication**. The server creates the authenticated session by doing two things. First, the server creates and stores a session object on the server's session store, such as memory, database or file. The session is like a secure box and has its own identifier session key which can open the box and retrieve the information back from the secure box. Secondly, the server sends the session key to the client and creates a cookie to keep the session key. On the following requests the web browser attaches the cookie automatically and then the server can get the information from the secure box by the session key cookie.

Because this approach is relied on the automatically sent cookie by web browser, there are some problems:

- **stateless and Server side scalability**

    The [RESTful API should be stateless](http://en.wikipedia.org/wiki/Representational_state_transfer#Stateless). There is no login or logout need to keep a session store on the API server to store the client state. The RESTful API server can scale easily.

    [See more here](http://stackoverflow.com/questions/3105296/if-rest-applications-are-supposed-to-be-stateless-how-do-you-manage-sessions)


- **CSRF/XSRF** [Cross-site request forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery)

    Since the traditional way is sending the session key by cookie, we need to deal with CSRF


- **Mobile**

    If we want to make our API work on not only web browser but also other native applications, cookie is not a good choice for native applications.


- **CORS**

    Let's say your front-end application is hosted on http://app.example.com and the API server is hosted on http://api.example.com

    Because of the *limit third party cookie policy*, by most of the browser's default settings, the Javascript hosted on app.example.com can't access the cookies set for api.example.com.


#Token-Based Authentication

To solve these issues of Cookie-Based Authentication we have a better solution which is Token-Based Authentication.

Here are some great articles about Token-Based Authentication, you definitely should check them out.

- [Cookies vs Tokens. Getting auth right with Angular.JS](https://auth0.com/blog/2014/01/07/angularjs-authentication-with-cookies-vs-token/)

- [10 Things You Should Know about Tokens](https://auth0.com/blog/2014/01/27/ten-things-you-should-know-about-tokens-and-cookies/#token-size)


## JSON Web Token

>> JSON Web Token (JWT) is a compact URL-safe means of representing claims to be transferred between two parties. The claims in a JWT are encoded as a JSON object that is digitally signed using JSON Web Signature (JWS). IETF


The JWT token contains all the information for keeping the session between the client and server, such as user id. There is no session data stored on the server so it's stateless.

The JWT token is not sent alone with the cookie, so you don't need to worry about CSRF and CORS.

The JWT authentication works good with RESTful API, doesn't rely on cookie, it works good with mobile app.

### The JWT Specification


Here's an example:

<span style="color:#859900;">eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.</span><span style="color:#268bd2;">eyJzdWIiOjEyMzQ1Njc4OTAsIm5hbWUiOiJKb2huIERvZSIsImFkbWluIjp0cnVlfQ</span>.<span style="color:#dc322f">eoaDVGTClRdfxUZXiPs3f8FmJDkDE_VCQFXqKxpLsts</span>

The JWT token is divided into 3 parts by `.`:

1. A header
2. A payload
3. A signature

You can decode it at [jwt.io](http://jwt.io).

#### 1. The Header

The header `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9` after BASE64 decoding:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

`typ` is always "JWT" and the `alg` is the hash algorithm used to sign the token. Usually it's HMAC SHA-256.


#### 2. The Payload

The payload `eyJzdWIiOjEyMzQ1Njc4OTAsIm5hbWUiOiJKb2huIERvZSIsImFkbWluIjp0cnVlfQ` after BASE64 decoding:

```json
{
  "sub":1234567890,
  "name":"John Doe",
  "admin":true
}
```

You can store any information you want for keeping the session between the client and the server. For the APIs need the authorization, you need to send the token in the Authorization header, so watch out the size.

#### 3. The Signature

The signature `eoaDVGTClRdfxUZXiPs3f8FmJDkDE_VCQFXqKxpLsts` is signed based on the Payload, the Header, the secret key, and the `alg` in header.

>You can read more about the Json Web Token [specification](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html).
