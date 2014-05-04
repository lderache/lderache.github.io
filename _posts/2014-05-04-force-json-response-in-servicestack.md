---
layout: post
title: "Force json response in ServiceStack"
modified: 2014-05-04 01:57:37 -0400
#category: []
tags: [ServiceStack]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

I recently ran into an issue integrating a ServiceStack service with 3rd party software that needed to consume Json.  
<br />
As the 3rd party was not able to generate the http header **Accept: application/json** to let ServiceStack auto generate proper content I had to force it in the web service response format:


{% highlight c# %}
		var serviceResponse =  new UpdateDisplayArrowServiceResponse
                {
                    Status = SerialResponse.Status
                };

                return new HttpResult(serviceResponse, "application/json");
{% endhighlight %}

To see all supported formats in ServiceStack it is [here](https://github.com/ServiceStack/ServiceStack/wiki/Formats)
