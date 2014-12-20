JWT with Beego
=========

JWT authentication is such a common feature for Beego API applications, so it's better to make it a reusable module.

Let's create it as a Beego plugin.

1. Create the plugin folder under the `GOPATH`. I named it as `jwt`

2. Create the `jwt.go` file.

```go
package jwt

import (
    goJwt "github.com/dgrijalva/jwt-go"
)
```

I will use [`github.com/dgrijalva/jwt-go`](http://github.com/dgrijalva/jwt-go) as the basic JWT lib. So just install `jwt-go` by running `go get github.com/dgrijalva/jwt-go`.


3. I will use `HMAC SHA-256` to sign the JWT, it requires the RSA private and public key. Let's generate them:

`openssl genrsa -out jwt.rsa 1024
`openssl rsa -in demo.rsa -pubout > jwt.rsa.pub`

4. Let's create the Beego Controller for JWT:

```go
// oprations for Jwt
type JwtController struct {
	beego.Controller
}

func (this *JwtController) URLMapping() {
	this.Mapping("IssueToken", this.IssueToken)
}

// @Title IssueToken
// @Description Issue a Json Web Token
// @Success 200 string
// @Failure 403 no privilege to access
// @Failure 500 server inner error
// @router /issue-token [get]
func (this *JwtController) IssueToken() {
	this.Data["json"] = CreateToken()
	this.ServeJson()
}

func CreateToken() map[string]string {
	token := goJwt.New(goJwt.GetSigningMethod("RS256")) // Create a Token that will be signed with RSA 256.
	token.Claims["ID"] = "This is my super fake ID"
	token.Claims["exp"] = time.Now().Unix() + 36000
	// The claims object allows you to store information in the actual token.
	tokenString, _ := token.SignedString(RSAKeys.PrivateKey)
	// tokenString Contains the actual token you should share with your client.
	return map[string]string{"token": tokenString}
}
```


1. Install the `jwt-go` package:

`go get github.com/dgrijalva/jwt-go`


2. Generate the controller:

`bee generate controller jwt`


3. Not sure about the purpose:

`openssl genrsa -out demo.rsa 1024 # the 1024 is the size of the key we are generating`
`openssl rsa -in demo.rsa -pubout > demo.rsa.pub`

4. Create a configs package:

- configs/configs.go


```
package configs

import (
  "io/ioutil"
)


var (
  PrivateKey []byte
)


func Init() {
  PrivateKey, _ = ioutil.ReadFile("conf/demo.rsa")
}
```


5. Added the router:

```
	ns := beego.NewNamespace("/v1",

    ...........

    beego.NSNamespace("/jwt",
			beego.NSInclude(
				&controllers.JwtController{},
			),
		),

    ........

	)
```

6. The controller Jwt

```
package controllers

import (
  "time"
  jwt "github.com/dgrijalva/jwt-go"
	"github.com/astaxie/beego"

	"beeblog/configs"
)

type JwtController struct {
	beego.Controller
}

func (this *JwtController) URLMapping() {
	this.Mapping("IssueToken", this.IssueToken)
}

// @Title IssueToken
// @Description Issue a Json Web Token
// @Success 200 string
// @Failure 403 no privilege to access
// @Failure 500 server inner error
// @router /issue-token [get]
func (this *JwtController) IssueToken() {
  token := jwt.New(jwt.GetSigningMethod("RS256")) // Create a Token that will be signed with RSA 256. 
  token.Claims["ID"] = "This is my super fake ID"
  token.Claims["exp"] = time.Now().Unix() + 36000
 
  // The claims object allows you to store information in the actual token.
  tokenString, _ := token.SignedString(configs.PrivateKey)
 
  // tokenString Contains the actual token you should share with your client.
  this.Data["json"] =  map[string]string{"token": tokenString}
	this.ServeJson()
}
```

7. The CORS

[Enable CORS for beego](https://godoc.org/github.com/astaxie/beego/plugins/cors)

7. The front-end

I am using this Yeoman generator:

[generator-gulp-angular](https://github.com/Swiip/generator-gulp-angular)

Install [Angular JWT](https://github.com/auth0/angular-jwt):

`bower install angular-jwt --save`


The CORS OPTIONS issue:
![](http://www.html5rocks.com/static/images/cors_server_flowchart.png?raw=true)


Resources:

[angular-jwt](https://github.com/auth0/angular-jwt)

[Json Web Token](http://angular-tips.com/blog/2014/05/json-web-tokens-examples/)

[Generator Gulp Angular](https://github.com/Swiip/generator-gulp-angular)

[Angularjs Style Guide](https://github.com/johnpapa/angularjs-styleguide)


>> JSON Web Token (JWT) is a compact URL-safe means of representing claims to be transferred between two parties. The claims in a JWT are encoded as a JSON object that is digitally signed using JSON Web Signature (JWS). IETF


Here is the Json Web Token [specification](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html).
