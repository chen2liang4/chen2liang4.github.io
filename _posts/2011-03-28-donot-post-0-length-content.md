---
title: Do not post 0 length content
---

When reading this blog about post http://blog.zhaojie.me/2011/03/html-form-file-uploading-programming.html, I recalled one issue we encountered. We got unexpected response from IIS, that said 411 Length Required, after posting a AJAX request on Android.

A typical post package should look like this:
```
POST http://www.baidu.com/ HTTP/1.1
Host: www.baidu.com
Content-Length: 74
Content-Type: application/x-www-form-urlencoded
myText1=hello+world&myText2=%E4%BD%A0%E5%A5%BD%E4%B8%96%E7%95%8C
```
the content body is “myText1=hello+world&myText2=%E4%BD%A0%E5%A5%BD%E4%B8%96%E7%95%8C”, and Content-Length specifies its length.  
The code snippet below is from our project:
```javascript
function sendMessage(playerId, messageId, content) {
    //...
    jQuery.ajax({
        url: path,
        data: {'MessageId': messageId, 'Content': content},
        type: 'post'
        //....
    });
}
```
It works well on iPhone/Android 2.x, but fail on Android 1.5 in the case that messgeId and content are empty. When posting the content which length is 0, JS engine on iPhone/Android 2.x add this in package header – “Content-Length: 0”, but JS engine on Android 1.5 ignore this field. So IIS 7 refuses this post request with responding  411 error.

411 Length Required   
    The request did not specify the length of its content, which is required by the requested resource
 
So when posting 0 length content, please consider get method. Or adding some meaningless content, but it’s an ugly approach.