---
layout: post
title: Document your API using Swagger
date: 2014-10-24 15:43
author: arenhage
comments: true
categories: [Home, Java, jaxrs, rest, swagger, swagger-ui]
---
<a href="http://swagger.io/" title="Swagger" target="_blank">Swagger</a> in all it's simplicity is a framework used to represent and describe your Restful API's.
 
<strong>Why would we want to use Swagger?</strong>
Swagger is a good tool used for closing the gap between documentation and implementation when developing our RESTful API's. Used to its full extent, we get a nice sense of control given that we are able to explicitly declare both what there server expects to receive being called, as well as when the request have been handled, what we expect to to return.

<!--more-->

Swagger also provides an additional way of simplifying and describing our API with Swagger-ui, which adds a graphical user interface based on our swagger configuration. This provides us with not only with a good overview of our specification, but some really intuitive handling for test purposes.

<strong>Swagger in Action</strong>
Swagger supports a wide variety of languages and frameworks. The full specification can be found here: https://github.com/wordnik/swagger-spec

To illustrate how Swagger looks and how we can work with it, In this example I'm referring to a small sample application i have created that can be found here: https://github.com/arenhage/jersey_swagger_jaxrs_sample. The application is a "Swagger JAX-RS" application running on a jersey server utilizing Swagger-ui.

Sample
===========
<strong>UserResource.java</strong>

```java
@Path("/user")
@Api(value = "/user", description = "API regarding Users")
@Produces({"application/json", "application/xml"})
@Path("/user")
@Api(value = "/user", description = "API regarding Users")
@Produces({"application/json", "application/xml"})
public class UserResource {
  UserData userData = new UserData();
  @GET
  @Path("/{userId}")
  @ApiOperation(value = "Get user by Id", notes = "Get User By Id", response = User.class)
  @ApiResponses(value = {
    @ApiResponse(code = 400, message = "Invalid Id supplied"),
    @ApiResponse(code = 404, message = "User not found")
  })
  public Response getUserById(
    @ApiParam(value = "ID of user to fetch", required = true)
    @PathParam("userId") Long userId) throws WebApplicationException {
      User user = UserData.getUserById(userId);
      if(user != null) return Response.ok(UserData.getUserById(userId)).build();
      else return Response.status(Status.BAD_REQUEST).build();
    }
}
```

Swagger will render the fundamental documentation regarding your model objects based on two types of information, your resources and your models.
Bottoms up we have a Jersey Server exposing a RESTful application using JAX-RS together with Swagger describing its purpose and functionality. For those who have used javax.rs we notice that it is really straight forward having annotated the base path using javax.rs annotation <strong>@Path</strong> to define our relative url to this resource, together with what we expect to receive back from the server using the @Produces annotation. In order for Swagger to declare this as a Swagger resource, we also annotate this resource with the @Api (Swagger will only consider resources annotated with @Api). Having declared it as a Swagger resource, Swagger will feed of any other information that might be exposed by either further swagger annotations along with javax.rs annotations providing information about this resource.

The method <em>getUserById</em> is marked as a @GET method along with Swagger annotations describing more descriptive details e.g. @ApiOperation which adds some descriptive information regarding this specific method. It also marks the response for this method as <em>response = User.class</em>. This tells Swagger that this method will produce a response based on the User model:

```java
@XmlRootElement(name = "User")
public class User {
	
  public User() {
  }

  public User(long id, String username, String firstName, String lastName,
    String email, String password) {
    super();
    this.id = id;
    this.username = username;
    this.firstName = firstName;
    this.lastName = lastName;
    this.email = email;
    this.password = password;
  }

  private long id;
  private String username;
  private String firstName;
  private String lastName;
  private String email;
  private String password;

  @XmlElement(name = "id")
  public long getId() {
		return id;
	}
.
.
.
```

Some vital and interesting parts is the usage of @Apiparam along with @PathParam
```java
public Response getUserById(
  @ApiParam(value = "ID of user to fetch", required = true)
  @PathParam("userId") Long userId)
```

Just looking at the the method we can clearly deduct that the method will return a Response, and that response will be of type User(based on the prior information). The method expects a parameter <em>userId</em> and that parameter is required. From the work we have put in so far, lets start the server and look at how our documentation is interpreted by Swagger:

```json
{
  "apiVersion": "1.0.0",
  "swaggerVersion": "1.2",
  "basePath": "http://localhost:8002/jersey-swagger/api",
  "resourcePath": "/user",
  "produces": [
      "application/json",
      "application/xml"
  ],
  "apis": [
      {
          "path": "/user/{userId}",
          "operations": [
              {
                  "method": "GET",
                  "summary": "Get user by Id",
                  "notes": "Get User By Id",
                  "type": "User",
                  "nickname": "getUserById",
                  "parameters": [
                      {
                          "name": "userId",
                          "description": "ID of user to fetch",
                          "required": true,
                          "type": "integer",
                          "format": "int64",
                          "paramType": "path"
                      }
                  ],
                  "responseMessages": [
                      {
                          "code": 400,
                          "message": "Invalid Id supplied"
                      },
                      {
                          "code": 404,
                          "message": "User not found"
                      }
                  ]
              }
          ]
      }
  ],
  "models": {
      "User": {
          "id": "User",
          "properties": {
              "id": {
                  "type": "integer",
                  "format": "int64"
              },
              "username": {
                  "type": "string"
              },
              "password": {
                  "type": "string"
              },
              "email": {
                  "type": "string"
              },
              "firstName": {
                  "type": "string"
              },
              "lastName": {
                  "type": "string"
              }
          }
      }
  }
}
```

The result is Swaggers raw interpretation of our RESTful API for the User resource. In Swagger-ui it looks like this:

<a href="{{ site.baseurl }}/images/2014-10-24-document-your-api-using-swagger/userresource.png"><img src="{{ site.baseurl }}/images/2014-10-24-document-your-api-using-swagger/userresource.png" alt="userresource" width="300" height="235" class="alignnone size-medium wp-image-171" /></a>

The UI provides a really nice usability in terms of testing and validating the functionality of our API, in the test below, we provide a valid <strong>userId</strong> and we state that we want to receive it as a json response:
 
<a href="{{ site.baseurl }}/images/2014-10-24-document-your-api-using-swagger/userresourcecall.png"><img src="{{ site.baseurl }}/images/2014-10-24-document-your-api-using-swagger/userresourcecall.png" alt="userresourcecall" width="300" height="261" class="alignnone size-medium wp-image-174" /></a>

In this example, if we would have provided a non-existent userId, the server would return a 400 and a no-content response. The way we handle responses is complete up to the programmer. We could have a handling where we construct our own Exceptions and have a more descriptive error response.

<strong>specification:</strong> [https://github.com/wordnik/swagger-spec](https://github.com/wordnik/swagger-spec)

<strong>samples:</strong> [https://github.com/swagger-api/swagger-core/tree/master/samples](https://github.com/swagger-api/swagger-core/tree/master/samples)

Other documentation frameworks that might be interesting:

* <a href="https://apiblueprint.org/" title="API Blueprint" target="_blank">API Blueprint</a>
* <a href="http://apiary.io/" title="apiary" target="_blank">apiary</a> (Blueprint flavour)
* <a href="http://raml.org/" title="RAML" target="_blank">RAML</a>
* <a href="http://www.mashery.com/product/io-docs" title="I/O Docs" target="_blank">I/O Docs</a>
