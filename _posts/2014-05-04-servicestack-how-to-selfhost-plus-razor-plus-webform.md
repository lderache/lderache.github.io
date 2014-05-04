---
layout: post
title: "ServiceStack how to: SelfHost + Razor + WebForm"
modified: 2014-05-04 09:01:00 -0400
tags: [ServiceStack]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

### TL;DR

Check out the example Github repo: 

### Why this how to ?

When I started with ServiceStack I wanted to develop a simple web app with these 3 components:

* SelftHost: It is great to just copy the output of the project and run it on Linux with Mono without hassle
* Razor: ServiceStack only without any other dependency
* Webform authentication: by far most common authentication method for simple web projects (eventually using SSL for security)

ServiceStack wiki is great and I quickly discovered the Razor Rockstars project that brings a lot of answers but it was, at first, not easy to see what I really needed to put together

### What we will be building

A simple web app retrieving car license plate numbers. This could be for example all car present in a parking that a registered user can access.
For simplicity of the code we will put user infos and sensor data directly in the code.

We will be doing things in this order:

1. Generate the basic web service to retrieve the data
2. Add html views
3. Add Authentication

### Get started : Self-Host

Create a console application and install ServiceStack from NuGet.

* Create your service and the DTOs

{% highlight c# %}

    // Basic car class
    public class Car
    {
        public string Plate { get; set; }
    }

    // Request DTO
    [Route("/Cars", "GET")]
    public class CarRequest
    {
    }

    // Response DTO
    public class CarResponse
    {
        public List<Car> Cars { get; set; }
    }

    public class CarService : Service
    {
        List<Car> Cars = new List<Car>
        {
            new Car { Plate = "FG98745" },
            new Car { Plate = "VN236PL" }
        };

        public object Get(CarRequest request)
        {
            return Cars;
        }
    }
{% endhighlight %}


* Then the AppHost and the main

{% highlight c# %}

 class Program
    {
        //Define the Web Services AppHost
        public class AppHost : AppHostHttpListenerBase
        {
            public AppHost()
                : base("HttpListener SelfHost Demo", typeof(CarService).Assembly) { }

            public override void Configure(Funq.Container container)
            {
            }
        }

        static void Main(string[] args)
        {
            LogManager.LogFactory = new ConsoleLogFactory();

            var listeningOn = args.Length == 0 ? "http://*:8090/" : args[0];

            var appHost = new AppHost();

            appHost.Init();
            appHost.Start(listeningOn);

            Console.WriteLine("AppHost Created at {0}, listening on {1}",
                DateTime.Now, listeningOn);

            Console.ReadKey();
        }
    }

{% endhighlight %}

### Creating Razor views

Install **ServiceStack.Razor** from NuGet. 

It is not installed by default with other ServiceStack packages.

Create a folder **Views** in your solution and put inside **_Layout.cshtml** and **cars.cshtml**.

Then put the an other pages called **Login.cshtml** at the root of the project, **outside the Views**.

The reason for that is that we want to be able to call the login page directly. The cars view will be rendered through the service call.

Check [this SO answer](http://stackoverflow.com/questions/13206038/servicestack-razor-default-page/13206221#13206221) for more details.

Select all *.cshtml file right click and see the properties then select **Copy if newer* so that the files will be copied in the project output during generation.

Input the following contents:

* _Layout.cshtml 

{% highlight html %}

<!doctype html>
<html>
<head>
    <title>Self-Host Razor Web Auth Demo</title>
</head>
<body>
<div class="container">
	@RenderBody()
</div>
</body>
</html>

{% endhighlight %}



* login.cshtml

{% highlight html %}

<div>
    <form role="form" action="/Auth/Credentials" method="post">
        <div>
            <label for="InputName">User</label>
            <div class="input-group">
                <input type="text" name="username" placeholder="Enter username" required>
            </div>
        </div>

        <div>
            <label for="InputPassword">Password</label>
            <div class="input-group">
                <input type="password" name="password" placeholder="Enter Password" required>
            </div>
        </div>

        <input type="hidden" name="continue" value="/Cars" />

        <input type="submit" name="submit" value="Enter" />
    </form>
</div>


{% endhighlight %}

We need to input the following config in App.config for Razor views

{% highlight html %}

<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <sectionGroup name="system.web.webPages.razor" type="System.Web.WebPages.Razor.Configuration.RazorWebSectionGroup, System.Web.WebPages.Razor, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35">
      <section name="host" type="System.Web.WebPages.Razor.Configuration.HostSection, System.Web.WebPages.Razor, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
      <section name="pages" type="System.Web.WebPages.Razor.Configuration.RazorPagesSection, System.Web.WebPages.Razor, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
    </sectionGroup>
  </configSections>
  <appSettings>
    <add key="webPages:Enabled" value="false" />
  </appSettings>
  <system.web>
    <httpHandlers>
      <add path="*" type="ServiceStack.WebHost.Endpoints.ServiceStackHttpHandlerFactory, ServiceStack" verb="*" />
    </httpHandlers>
    <compilation debug="true">
      <assemblies>
        <add assembly="System.Web.WebPages.Razor, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
      </assemblies>
      <buildProviders>
        <add extension=".cshtml" type="ServiceStack.Razor.CSharpRazorBuildProvider, ServiceStack.Razor" />
      </buildProviders>
    </compilation>
  </system.web>
  <!-- Required for IIS 7.0 -->
  <system.webServer>
    <handlers>
      <add path="*" name="ServiceStack.Factory" type="ServiceStack.WebHost.Endpoints.ServiceStackHttpHandlerFactory, ServiceStack" verb="*" preCondition="integratedMode" resourceType="Unspecified" allowPathInfo="true" />
    </handlers>
  </system.webServer>
  <system.web.webPages.razor>
    <host factoryType="System.Web.Mvc.MvcWebRazorHostFactory, System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
    <pages pageBaseType="ServiceStack.Razor.ViewPage">
      <namespaces>
        <add namespace="ServiceStack" />
        <add namespace="ServiceStack.Html" />
        <add namespace="ServiceStack.Razor" />
        <add namespace="ServiceStack.Text" />
        <add namespace="ServiceStack.OrmLite" />
        <add namespace="SelfHostRazorWebFormAuth" />
      </namespaces>
    </pages>
  </system.web.webPages.razor>
 
</configuration>

{% endhighlight %}


Now if you start the project and browse to http://localhost:8090/login you should see the login page !

And if you input some login and password the result will be:

{% highlight html %}
Handler for Request not found: 


Request.HttpMethod: POST
Request.PathInfo: /Auth/Credentials
Request.QueryString: 
Request.RawUrl: /Auth/Credentials
{% endhighlight %}

This is normal since we did not activate any Auth module yet :)

### Adding Auhtentication

Update your AppHost Configure method as follow:

{% highlight C# %}

 public override void Configure(Funq.Container container)
            {
                Plugins.Add(new RazorFormat());

                Plugins.Add(new AuthFeature(() => new AuthUserSession(),
                  new IAuthProvider[] { 
                    new CredentialsAuthProvider(), //HTML Form post of UserName/Password credentials
                  }));

                Plugins.Add(new SessionFeature());
            }

{% endhighlight %}

And if you try again to login, this time it is ConfigurationErrorsException which tells you that a repository is needed to authenticate.

Let's create an InMemory Repo with a user:

{% highlight C# %}

var userRep = new InMemoryAuthRepository();
                container.Register<IUserAuthRepository>(userRep);

                UserAuth userDemo = new UserAuth
                {
                    UserName = "demo"
                };

                userRep.CreateUserAuth(userDemo, "demo");

{% endhighlight %}

And now if you try to login with demo/demo it works !

The login.cshtml webform is posting the user/password to /Auth/Credentials which is registered by ServiceStack

Then the following input declare the continue variable used for the redirection after login.

{% highlight html %}

<input type="hidden" name="continue" value="/Cars" />

{% endhighlight %}

Still a few issues pending though. 

* We can access the service /cars directly. We need to avoid that
* We want to be able to logout
* We want to see the /cars result in our own template

##### Protect the service access

Just decorate the service with Authenticate attribute and you are done

{% highlight html %}
[Authenticate]
    public class CarService : Service
    {
	...
    }

{% endhighlight %}

##### Logout

Just call /auth/logout. It is as simple as that. A simple link is enough.

##### See the service result in our template

So far we did not touch the cars.cshtml. Let's create it:

{% highlight html %}

@inherits ViewPage<CarResponse>

<table>
    @foreach (var item in Model.Cars)
    {
        <tr>
            <td>@item.Plate</td>
        </tr>
    }
</table>

<a href="/auth/logout">logout</a>

{% endhighlight %}

Notice that we added the logout link.

### Final test

And that's it. We have web form authentication working with a Self-Host ServiceStack instance. We can render the Service response in Razor view with strong typed view model.






