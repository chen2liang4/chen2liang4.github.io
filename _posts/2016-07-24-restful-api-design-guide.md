---
title: RESTful API design guide
---

This guide documents guidelines, recommendations and best practices for designing a RESTful API, in order to provide helps for common decisions, issues in RESTful API design. These principles should be followed:
* Try to use big patterns to resolve issues, we don't know how the client consumes APIs.
* Be practical, it's allowed to be anti-patterns, but only when you don't have option.
* API design should allow for implementation, choose option which is best supported by your framework.
* This guide should be always WIP (work in process).

## URI
### Root path
Provide a root path for all APIs in case that API are mixed with web page or other web resources, e.g:
* https://api.example.com/ 
* https://example.com/api/

We also need to distinguish API and AJAX request. AJAX request is designed for web interaction, but API is designed for all possible clients. So DO NOT put AJAX request into API.

### Version
Use version if you can't guarantee API consumers be able to upgrade to the latest version you release, e.g:
* https://example.com/api/v1/ 
* https://example.com/api/latest/

### Path formats
Use downcased and dash-seperated path name, e.g:
* https://example.com/api/v1/users
* https://example.com/api/v1/user-types 

Regarding query parameters which often will be mapped to properties of a class, choose one style which is compatible with your framework, like camelCase.

## Authentication
Considering support basic authentication and cookie-based authentication both. Some framework like Spring makes this goal to be achieved conveniently.  
For mobile app, a cookie-based authentication is reasonable for developers, since there are many interactions after the user log in. see Why [HTTP Basic Auth is Bad](http://adrianotto.com/2013/02/why-http-basic-auth-is-bad/)  
For the API consumer which only call API few time, basic authentication has no login required then it make API calling more easily.

## Resource
Restful API is resource-centric, which means resource modeling is key works. We simply classify resource as
* entity resource
* action resource
* gateway resource.

### Entity Resource
Entity resource represents something or domain concept which has properties and usually could be identified, like user, customer, order etc. It reflects one business model or concept, sometimes could be mapped to the database table.

#### Naming
Use the plural version of a resource name unless the resource in question is a singleton within the system (for example, the overall status of the system might be /status). This keeps it consistent in the way you refer to particular resources.  
* https://example.com/api/v1/users
* https://example.com/api/v1/users/23
* https://example.com/api/v1/users/23/address 

#### Sub-resource
Sub-resource is owned by the parent, and addressed through parent id.
* https://example.com/api/v1/users/23/addresses
* https://example.com/api/v1/departments/3/teachers/53 

Sub-resource should have N:1 relation with the parent.  
Object relations with sub-resource:
* **Composition**. whole and part relation, the parent owns child, and they have same life cycle. when the resource is deleted, the sub-resource will disappear too. E.g, user and address, house and room. Resources with composition relation should be sub-resource
* **Aggregation**. The resource own sub-resource, but they have separate life cycle. E.g, department and teacher. The aggregation relation could be sub-resource, but has not to be. If path is too deep, the relation is not strong, the child will be accessed without parent frequently, we should consider not to use sub-resource.
* **Association**. No owner in association relation. Like teacher and student, a teacher teaches many student, but a student is taught by many teachers. So we can say a teacher owns a student, then the student is not sub-resource of the teacher.

Alternative approach for composite sub-resource is anchor, e.g.:  
* https://example.com/api/v1/users/23#address   

**NOTE** It's better to address one sub-resource only via one parent level. Be careful with sub-resource with deep hierarchy.
* https://example.com/api/v1/departments/3/teachers/54/address  

Sub-resource could be first resource.  
* https://example.com/api/v1/teachers/54 

### Non-CRUD actions
Ideal RESTful API only provides CRUD operations by HTTP methods. We have to escape some operations to CRUD through various approaches, like treating an operation as a new resource. We think that it's more practical to allow non-CRUD actions and put it in URL. The action should appended to resource URI  
* https://example.com/api/v1/articles/3/like  
* https://example.com/api/v1/users/23/password/reset   

Alternative convention is place a actions prefix to delineate resource  
* https://example.com/api/v1/articles/3/actions/like  
* https://example.com/api/v1/users/23/password/actions/reset   

However, do not use non-CRUD actions only because that it's simple for design. RESTful API is resource-centric, first we should consider if there is a resource suit for the special action, and if we need to create one. Only if it's hard to do that, or created resource is not easy to understand, then we will take the approach of non-CRUD actions.

### Action Resource
Action resource represents a business capabilities, complex operation or process. It's hard to be mapped to one entity resource, and usually it involves more than one entity resource. If only one entity resource involved, consider entity resource with non-CRUD action. Action resource can be tracked and related data could be persisted. Use nouns or verbs to name action resource.  
* https://example.com/api/v1/order-search?status=closed  
* https://example.com/api/v1/user-entrollments  
* https://exmaple.com/api/v1/refund

As entity resource with non-CRUD action, we also be careful with the design of action resource, in case that a resource-centric architecture evolve to an action-centric one, then we'll lose many advantages.

#### Non-CRUD actions vs. Action Resource
These two are alikely, e.g.:
* https://example.com/api/v1/order-search?status=closed   
* https://example.com/api/v1/orders/search?status=closed   

Action Resource has same level with the resource which Non-CRUD actions append to.   
And Non-CRUD actions usually response the resource that it append to.

### Gateway Resource
Entity resource and action resource possess atomic. Gateway resource work as on facade of these entity resources and action resources. We can design a gateway resource for one long process which involves couple of resource requests, or for reducing interactions between API client and server (e.g.: need to get many resources immediately after posting one request for rendering one UI page).  
* https://example.com/api/v1/homepage   

Avoiding to design API for UI request. Otherwise, we'll lose reusability. The architect should mainly be comprised by entity resources, then action resources, then gateway resource.

## Request
Use HTTP standard methods below, JSON payload and accept JSON by default.
### HTTP methods
  
| **Verb** | **Usage** | **Safe** | **Idempotent** | **Notes** |
|----------|-----------|----------|----------------|-----------| 
| GET      | Read      | Yes      | Yes            | All safe operation should use GET method |
| POST     | Create    |          |                | Create a new resource |
| PUT      | Update    |          | Yes            | Update a resource |
| PATCH    | Update    |          | Yes            | Update part of a resource |
| DELETE   | Delete    |          | Yes            | Delete a resource |

For safe operations, use GET methods then benefit from HTTP features, like cache.

### Projection
Use projection to indicate what fields in the response for reducing payload, use 'self' to get all fields except associated data. e.g.:  
```json
GET /orders/234
Response 
{ 
    "id": "234",
    "code": "BS023sxE", 
    "customer": { 
        "id": "323",
        "name" "neo" 
    },
    "totalPrice": 100, 
    "shippingCost": 9, 
    "productCount": 1, 
    "items": [{ 
        "id": "234", 
        "product": { 
            "id": "923", 
            "name": "Cofee", 
            "brand": "Zhou-buck" 
        }, 
        "quantity": 3, 
        "price": 100 
        }], 
        "status": "CLOSED", 
        "createdAt": "2016-8-29T21:12:00Z", 
        "updatedAt": "2016-8-29T21:12:00Z" } 
        
GET /orders/234?fields=totalPrice,shippingCost 
Response 
{
    "totalPrice": 100, 
    "shippingCost": 9 
} 

GET /orders/234?fields=self 
Response 
{ 
    "id": "234", 
    "code": "BS023sxE", 
    "customer": { 
        "id": "323" 
    }, 
    "totalPrice": 100, 
    "shippingCost": 9, 
    "status": "CLOSED", 
    "createdAt": "2016-8-29T21:12:00Z", 
    "updatedAt": "2016-8-29T21:12:00Z" 
} 
```
### Expansion
In order to simplify responses, API only return parts of resource by default sometime. Use expansion to request additional fields of resource, use 'all' keyword to get full resource.
```json
GET /orders/234 
Response 
{ 
    "id": "234", 
    "code": "BS023sxE", 
    "customer": { 
        "id": "323" 
    }, 
    "totalPrice": 100, 
    "status": "CLOSED", 
    "createdAt": "2016-8-29T21:12:00Z", 
    "updatedAt": "2016-8-29T21:12:00Z" 
} 

GET /orders/234?expand=customer,shippingCost 
Response 
{ 
    "id": "234", 
    "code": "BS023sxE", 
    "customer": { 
        "id": "323", 
        "name": "neo" 
    }, 
    "totalPrice": 100, 
    "shippingCost": 9, 
    "status": "CLOSED", 
    "createdAt": "2016-8-29T21:12:00Z", 
    "updatedAt": "2016-8-29T21:12:00Z" 
} 

GET /orders/234?expand=all 
Response 
{ 
    "id": "234", 
    "code": "BS023sxE", 
    "customer": { 
        "id": "323", 
        "name": "neo" 
    }, 
    "totalPrice": 100, 
    "shippingCost": 9, 
    "productCount": 1, 
    "items": [{ 
        "id": "234", 
        "product": { 
            "id": "923", 
            "name": "Cofee", 
            "brand": "Zhou-buck" 
        }, 
        "quantity": 3, 
        "price": 100 
    }], 
    "status": "CLOSED", 
    "createdAt": "2016-8-29T21:12:00Z", 
    "updatedAt": "2016-8-29T21:12:00Z" 
} 
```

### Filtering
Use filtering to reduce resource number of response.  
* GET /orders?user=234&status=closed   
* GET /ptos/search?user=234&status=closed  

### Sorting
Sorting is supported in the sort query parameter, by default it's ascending. To specify the sorting use "-" or "+" sign. e.g.:  
* GET /orders?user=234&sort=+name GET /orders?user=234&sort=+name,-createdtime   

Alternative approach use "sort" and "order" two query parameters, e.g.:  
* GET /orders?user=234&sort=name&order=asc  
* GET /orders?user=234&sort=name,createdtime&order=asc,desc 

### Paging
Use paging to limit the response size for resources that return a potentialy large collection of items. A request to a paged API will result in a values array wrapped in a JSON object.  
```json
GET /orders/search?user=234&page=2&pageSize=20
Response 
{ 
    "page": 2, 
    "pageSize": 20, 
    "total": 256, 
    "items": [{}, {}] 
} 
```
Return paged response by default for potential bulk data request
```json
GET /orders 
Response 
{ 
    "page": 1, 
    "pageSize": 20, 
    "total": 256, 
    "items": [{}, {}] 
} 
```
### Cache
HTTP cache
* [Squid](http://www.squid0-cache.org/)
* [Traffic Server](http://incubator.apache.org/projects/trafficserver.html)
These cache server improves performance without touching code, but API should set expires and Cache-control response header well.

### Concurrency
Use ETag and last modified to validate if the resource has changed for avoiding concurrency issue.

### Batch request
Avoiding to create gateway resources for each scenario that needs to reduce interaction between clients and server, batch request provides a general solution.
```json
POST /batch-request 
[
    { "method": "POST", "url": "users", "payload": { "name": "neo" } }, 
    { "method": "GET", "url": "users/234" }
] 
Response 
[ 
    { "statusCode": 201, "payload": { user attributes } }, 
    { "statusCode": 200, "payload": { user attributes } } 
] 
```

### Asynchronous request
When the sever receives one asynchronous request, an asynchronous task resource is created and return to the client immediately. The client can get status, progress etc. of this request via this resource.
```json
POST /payments 
{ .... } 
Response
202 Accepted 
{ 
    "id": "3", 
    "status": "in-progress", 
    "progress": "50%", 
    "createdAt": "2016-8-29T21:12:00Z", 
    "updatedAt": "2016-8-29T21:12:00Z" 
} 

GET /payment-tasks/3 
Response 
200 OK 
{ 
    "id": "3", 
    "status": "closed", 
    "progress": "100%", 
    "createdAt": "2016-8-29T21:12:00Z", 
    "updatedAt": "2016-8-29T22:10:00Z", 
    "receipt": { 
        "id": "23", 
        "user": {"id": "234"}, 
        "amount": 100, 
        "currency": "RMB", 
        ..... 
    }
} 
```
## Response
### HTTP Status Code
Only use well-known http status code
#### 200 OK
general success response if no other appropriate 2XX code
#### 201 Created
new resource is created
#### 202 Accepted
Asynchronous call
#### 204 No Content
success, but no content. Usually use for deletion
#### 400 Bad Request
the client breaks API contract, such as no value for required filed, wrong data type etc.
#### 401 Unauthorized
Return if current user is not authorized( not sign in)
#### 403 Forbidden
Return if current user has no permission
#### 404 Not Found
Return if the resource not found
#### 409 Conflict
Return if business rule fails, such as password don't match rule, the user cannot be deleted due to having orders etc.
#### 500 Server error
The server fails

Decision [TBD] if involving business violation. like not found resource in the request. E.g, post /transaction/transfer, but can't find receiver. This should be 404 or 409.

| **Request Method** | **Scenario**                    | **Response Status** | **Response Payload** |
|--------------------|---------------------------------|---------------------|----------------------|
| GET                | retrieve resource               | 200                 | requested resource |
| GET                | long time retrieve resource     | 202                 | asynchronous task resource |
| POST               | request action/gateway resource | 200                 | corresponding resources or no content |
| POST               | create an entity resource       | 201                 | created resource |
| POST               | long time action                | 202                 | asynchronous task resource |
| PUT                | update a resource               | 200                 | updated resource |
| PATCH              | patially update a resource      | 200                 | updated resource |
| DELETE             | delete an exsiting resource     | 204                 | no content |

### API Errors
When any errors occurred (4xx or 500 status), provide detail in the body of the response beside status code in the following format:
```json
{ 
    "errorCode": "8900", 
    "message": "{description of error}", 
    "details": "{additional details}" 
} 
```

### Resource Representation
Use JSON to represent resource data. Response of resource request usually is representation of entity resource, like getting a user, response is a user data in JSON.   
It also works with action resource and gateway resource, like transferring money, response is updated account data.   
Sometime, it's hard to represent result by entity response or collection response. So it allows to create result resource as response.
```json
GET /ptos/search?user=234 
Response 
[{ 
    "ptoId": "2323", 
    "user": "neo", 
    "projectManager": "Guru",
    "leaveType": "annual leave"
}] 
```
This only return user name and project manager name, then we treats this as a result resource, instead of entity resource.
#### Full resource as possible
Always provide full resource representation as possible, including collections, linked resource.
```json
{ 
    "orderId": "2323", 
    "user": { 
        "id": "2346", 
        "name": "neo" 
    }, 
    "items": [{ 
        "id": "3,", 
        "product": { 
            "id": "3434", 
            "name": "coffee", 
            "brand": "zhou-buck" 
        }, 
        "quantity": 3, 
        "price": 863.4 
    }] 
} 
```
Based on needs, decide nest hierarchy. For example, only return product id comparing the example upper.
```json
{ 
    "orderId": "2323",
    "user": { 
        "id": "2346", 
        "name": "neo" 
    },
    "items": [{ 
        "id": "3", 
        "product": { 
            "id": "3434" 
        }, 
        "quantity": 3, 
        "price": 863.4 
    }]
} 
```
Alternative is to return ids of linked resource.
```json
{ 
    "id": "1234567", 
    "name": "demoapp", 
    "owner": { "id": "456789", ...}, 
    "issues": [{ "id": "833" }, { "id": "834" }] ... } 
```
#### Provide Href resource
Considering return URL of resource for address convenience.
```json
{ 
    "id": "1234567", 
    "name": "demoapp", 
    "owner": { 
        "id": "456789", 
        "href": "/users/456789" 
    }, 
    "issues": [
        { "id": "833", "href": "/issues/833" }, 
        { "id": "834", "href": "/issues/833", }
    ] 
    ... 
} 
```
#### Nest linked resource
Represent linked resource with a nested object, e.g.:
```json
{ 
    "name": "service-production", 
    "owner": { 
        "id": "4533" 
    }} 
```
Instead of an value, e.g.:
```json
{ 
    "name": "service-production", 
    "ownerId": "4533" 
} 
```
This approach makes it possible to inline more information about the related resource without having to change the structure of the response or introduce more top-level response fields, e.g.:
```json
{ 
    "name": "service-production", 
    "owner": { 
        "id": "4533", 
        "name": "Neo", 
        "email": "neo@example.com" 
    }
} 
```
#### Provide timestamps for resource
Provide timestamps for the resource by default, then make it possible for caching. 
```json
{
    ... 
    "createdAt": "2012-01-01T12:00:00Z", 
    "updatedAt": "2012-01-01T12:00:00Z"
    ... 
}
```

#### Use string type for id
Considering NO-SQL and distributed db, using string type to represent id, even it's an auto increased field in the db.

## JSON style
Refer to Google JSON Style Guide Here are some examples:
* Property names must be camel-cased, ascii strings
* Array types should have plural property names
* Consider removing empty or null values
* Enum values should be represented as strings
* Dates should be formatted as recommended by RFC 3339

## Security
Use SSL by default  
Also refer to [REST Security Cheat Sheet](https://www.owasp.org/index.php/REST_Security_Cheat_Sheet)

## Tools
### Documents
* [RAML](http://raml.org/), With RAML and its tools to design and document API, such as [API Workbench](http://apiworkbench.com/), [API Console](https://github.com/mulesoft/api-console)
* [Swagger](http://swagger.io/)

### JSON validator
TBD

## References
* [RESTful Web Services Cookbook](https://www.amazon.com/RESTful-Web-Services-Cookbook-Scalability/dp/0596801688/ref=sr_1_1?ie=UTF8&qid=1472461353&sr=8-1&keywords=RESTful+Web+Services+Cookbook)
* [Github API](https://developer.github.com/v3/)
* [Paypal API Standards](https://github.com/paypal/api-standards)
* [Paypal REST APIs](https://developer.paypal.com/docs/api)
* [JIRA 6.1 REST API documentation](https://developer.atlassian.com/static/rest/jira/6.1.html)
* [Saleforce API](http://releasenotes.docs.salesforce.com/en-us/summer15/release-notes/rn_api.htm)
* [HTTP API Design Guide](https://geemus.gitbooks.io/http-api-design/content/en/)
* [REST API Design - Resource Modeling](https://www.thoughtworks.com/insights/blog/rest-api-design-resource-modeling)
* [Restful API 的设计规范](http://novoland.github.io/%E8%AE%BE%E8%AE%A1/2015/08/17/Restful%20API%20%E7%9A%84%E8%AE%BE%E8%AE%A1%E8%A7%84%E8%8C%83.html)
* [RESTful HTTP的实践](http://www.infoq.com/cn/articles/designing-restful-http-apps-roth)
* [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md)
* [Google JSON Style Guide](https://google.github.io/styleguide/jsoncstyleguide.xml)