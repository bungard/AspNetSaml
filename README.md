# AspNetSaml

[![Build status](https://ci.appveyor.com/api/projects/status/j47v6dygoqpwknay?svg=true)](https://ci.appveyor.com/project/NumbersInternational/aspnetsaml)

Very simple SAML 2.0 "consumer" implementation in C#. It's a *SAML client* library, not a *SAML server*, allows adding SAML single-sign-on to your ASP.NET app, but *not* to provide auth services to other apps.

Consists of **one short C# file** you can throw into your project (or [install via nuget](#new-nuget)) and start using it. Originally forked from OneLogin's .NET SAML library, but we had to fix a lot of stuff...

## Usage

### How SAML works?

SAML workflow has 2 steps:

1. User is redirected to the SAML provider (where he authenticates)
1. User is redirected back to your app, where you validate the payload

Here's how you do it:

### 1. Redirecting the user to the saml provider:

```c#
//specify the SAML provider url here, aka "Endpoint"
var samlEndpoint = "http://saml-provider-that-we-use.com/login/";

var request = new AuthRequest(
	"http://www.myapp.com", //put your app's "unique ID" here
	"http://www.myapp.com/SamlConsume" //assertion Consumer Url - the redirect URL where the provider will send authenticated users
	);
	
//generate the provider URL
string url = request.GetRedirectUrl(samlEndpoint);

//then redirect your user to the above "url" var
//for example, like this:
Response.Redirect(url);
```

### 2. User has been redirected back

User is sent back to your app - you need to validate the SAML response ("assertion") that you recieved via POST.

Here's an example of how you do it in ASP.NET MVC

```c#
//ASP.NET MVC action method... But you can easily modify the code for Web-forms etc.
public ActionResult SamlConsume()
{
	//specify the certificate that your SAML provider has given to you
	string samlCertificate = @"-----BEGIN CERTIFICATE-----
BLAHBLAHBLAHBLAHBLAHBLAHBLAHBLAHBLAHBLAHBLAHBLAH123543==
-----END CERTIFICATE-----";

	Saml.Response samlResponse = new Response(samlCertificate);
	samlResponse.LoadXmlFromBase64(Request.Form["SAMLResponse"]); //SAML providers usually POST the data into this var

	if (samlResponse.IsValid())
	{
		//WOOHOO!!! user is logged in
		//YAY!
		
		//Some more optional stuff for you
		//lets extract username/firstname etc
		string username, email, firstname, lastname;
		try
		{
			username = samlResponse.GetNameID();
			email = samlResponse.GetEmail();
			firstname = samlResponse.GetFirstName();
			lastname = samlResponse.GetLastName();
		}
		catch(Exception ex)
		{
			//insert error handling code
			//no, really, please do
			return null;
		}
		
		//user has been authenticated, put your code here, like set a cookie or something...
		//or call FormsAuthentication.SetAuthCookie() or something
	}
}
```

# Dependencies

Project should reference `System.Security`

# Nuget

I've published this to Nuget.

`Install-Package Displayr.AspNetSaml`

This will add a reference to a compiled version of this assembly.

# Nuget Publishing Steps

1. Open the solution in Visual Studio
2. Do both a debug build
3. Copy the build output Saml.dll to the lib folder.
4. Update Saml.Nuspec version number, and please use the SEMVER (https://semver.org/) scheme to decide which digit to increment. Commit this and push.
5. Update Saml.Nuspec releasenotes fields and put some description of what you changed there. But don't commit this.
6. From the Package Manage Console, run this command:

nuget pack Saml\Saml.nuspec

7. This will generate a file like Displayr.AspNetSaml.1.1.0.nupkg in your solution root folder.
8. Visit https://www.nuget.org/packages/Displayr.AspNetSaml/ and login, and click upload.
9. Upload the nupkg file generated.
10. In the preview, enter https://raw.githubusercontent.com/Displayr/AspNetSaml/master/README.md for the doco url.
11. Click Submit

# About Displayr

[![Displayr](https://github.com/Displayr/AspNetSaml/blob/master/displayr_d.jpg?raw=true)](https://www.displayr.com)

Powerful business intelligence and online reporting for survey data. Now hiring...

Company: https://www.displayr.com  
Careers: https://www.displayr.com/careers/  
