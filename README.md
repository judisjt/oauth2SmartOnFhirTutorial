# Using Angular 2 for Oauth2 Authorization with the Smart on Fhir Sandbox

I just learned how to create a website that makes authorized get requests to the smart on fhir server and I wanted to make a tutorial on how to do that in Angular 2. 

## Getting Started

The main thing that you need to be able to start this project would be to have NPM installed and then to have the angular-cli installed.
You might want to look up what Oauth2 authorization is, but that would just be for understanding more than anything.

### Prerequisites

To install NPM go to https://www.npmjs.com/get-npm and it is a free javascript package manager that helps organize all of the packages that you will end up downloading. It it pretty easy to use and if you don't know how I would look up a quick tutorial on that as well.


### Installing

A step by step series of examples that tell you have to get a development env running

This is after you have npm downloaded and installed. You have to install the Angular-cli through npm. Write this in the command prompt.

```
npm install angular-cli
```

After Angular is installed you will want to create a new angular project. Write this in the command prompt as well.

```
ng new PROJECT_NAME
```

Once you have an angular project up and running you should register yourself with the fhir sandbox DSTU2 server so you can test apps

After you login to the smart on fhir sandbox go to registered apps and click on register manually

'''
The app launch URI is the first page your website/app goes to and the redirect URI is the landing page after fhir authenticates the user
'''

After you have a registered app create a static javascript launch.html page in your app folder

'''

'''

Create another static html page called afterlaunch.html

'''

'''

Add both of those files to the angular assets so when you ng serve a local hosted webpage angular knows to load those files

'''

'''

For the launch.html page you will want to include the launch code from the sample authorization page that shows an example

'''
    <!DOCTYPE html>
    <html>
        <head>
            <title>Simple Auth App - Launch</title>
            <script src="http://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
        </head>
    <body>
        Loading...
        <script>
        // Change this to the ID of the client that you registered with the SMART on FHIR authorization server.
        var clientId = "16cbfe7c-6c56-4876-944f-534f9306bf8b";
        
        // For demonstration purposes, if you registered a confidential client
        // you can enter its secret here. The demo app will pretend it's a confidential
        // app (in reality it cannot be confidential, since it cannot keep secrets in the
        // browser)
        var secret = null;    // set me, if confidential
        
        // These parameters will be received at launch time in the URL
        var serviceUri = getUrlParameter("iss");
        var launchContextId = getUrlParameter("launch");
        
        // The scopes that the app will request from the authorization server
        // encoded in a space-separated string:
        //      1. permission to read all of the patient's record
        //      2. permission to launch the app in the specific context
        var scope = [
                "patient/*.read",
                "launch"
            ].join(" ");
            
        // Generate a unique session key string (here we just generate a random number
        // for simplicity, but this is not 100% collision-proof)
        var state = Math.round(Math.random()*100000000).toString();
        
        // To keep things flexible, let's construct the launch URL by taking the base of the 
        // current URL and replace "launch.html" with "index.html".
        var launchUri = window.location.protocol + "//" + window.location.host + window.location.pathname;
        var redirectUri = launchUri.replace("launch.html","index.html");
        
        // FHIR Service Conformance Statement URL
        var conformanceUri = serviceUri + "/metadata"
        
        // Let's request the conformance statement from the SMART on FHIR API server and
        // find out the endpoint URLs for the authorization server
        $.get(conformanceUri, function(r){

            var authUri,
                tokenUri;
            
            var smartExtension = r.rest[0].security.extension.filter(function (e) {
               return (e.url === "http://fhir-registry.smarthealthit.org/StructureDefinition/oauth-uris");
            });

            smartExtension[0].extension.forEach(function(arg, index, array){
              if (arg.url === "authorize") {
                authUri = arg.valueUri;
              } else if (arg.url === "token") {
                tokenUri = arg.valueUri;
              }
            });
            
            // retain a couple parameters in the session for later use
            sessionStorage[state] = JSON.stringify({
                clientId: clientId,
                secret: secret,
                serviceUri: serviceUri,
                redirectUri: redirectUri,
                tokenUri: tokenUri
            });

            // finally, redirect the browser to the authorizatin server and pass the needed
            // parameters for the authorization request in the URL
            window.location.href = authUri + "?" +
                "response_type=code&" +
                "client_id=" + encodeURIComponent(clientId) + "&" +
                "scope=" + encodeURIComponent(scope) + "&" +
                "redirect_uri=" + encodeURIComponent(redirectUri) + "&" +
                "aud=" + encodeURIComponent(serviceUri) + "&" +
                "launch=" + launchContextId + "&" +
                "state=" + state;
         }, "json");
        
        // Convenience function for parsing of URL parameters
        // based on http://www.jquerybyexample.net/2012/06/get-url-parameters-using-jquery.html
        function getUrlParameter(sParam)
        {
            var sPageURL = window.location.search.substring(1);
            var sURLVariables = sPageURL.split('&');
            for (var i = 0; i < sURLVariables.length; i++) 
            {
                var sParameterName = sURLVariables[i].split('=');
                if (sParameterName[0] == sParam) {
                    var res = sParameterName[1].replace(/\+/g, '%20');
                    return decodeURIComponent(res);
                }
            }
        }
        </script>
    </body>
</html>
'''

## Running the tests

Explain how to run the automated tests for this system

### Break down into end to end tests

Explain what these tests test and why

```
Give an example
```

### And coding style tests

Explain what these tests test and why

```
Give an example
```

## Deployment

Add additional notes about how to deploy this on a live system

## Built With

* [Dropwizard](http://www.dropwizard.io/1.0.2/docs/) - The web framework used
* [Maven](https://maven.apache.org/) - Dependency Management
* [ROME](https://rometools.github.io/rome/) - Used to generate RSS Feeds

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Billie Thompson** - *Initial work* - [PurpleBooth](https://github.com/PurpleBooth)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Hat tip to anyone who's code was used
* Inspiration
* etc
