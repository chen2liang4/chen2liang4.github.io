---  
title: ASP.NET MVC URL Conventions  
---  
  
1. Get request doesn't change status; Post request changes status.    
Bad:  
post product/query  
     name=xxx  
Good:  
get  product/query?name=xxx  
   
2. Url pattens is {controller}/{action}/{id}. The controller could comprise multiple segments, such as "Account/Setting"; action and Id only have one segment.    
  
3. One controller should focus on one resource, and id must identify it uniquely.    
Bad:  
product/detail/{row}/{column}  
even combination of row and column could identify one product, but don't use like this.  
Good:  
product/detail/1  
  
4. If one request specifies one resource, url should include id.    
Bad:  
post product/setprice  
     id=1&price=12.3  
Good:  
post product/setprice/1  
     price=12.3  
  
5. If return format of one action is multiple(JSON HTML, XML), specify it in url paramter; If not, no need return type parameter.     
example:  
get  product/detail/1?returnType=xml  
  
6. In a get request, parameters of the action should be url parameter; In a post request, parameters of the action should be in post body.    
Bad:  
get  product/query/abc  
post product/order/1/20  
good:  
get  product/query?name=abc  
post product/order/1  
     quantity=20  
  
7. Examples:    
**post product**  
create a product  
**get  product/detail/1**  
get details of the product of which id is 1.  
**post product/edit/1**  
update the product of which id is 1.  
**post product/delete/1**  
delete the product of which id is 1.  
**get  product/new**  
return a form for creating a new product.  
**get  product/show/1**  
return a HTML view for showing details of a product.  
**get  product/edit/1**  
return a form for editing a product.  
**get  product/delete/1**  
return a form for deleting a product.​  