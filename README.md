# Using Angular 2 for Oauth2 Authorization with the Smart on Fhir Sandbox

I just learned how to create a website that makes authorized get requests to the smart on fhir server and I wanted to make a tutorial on how to do that in Angular 2. 

## Getting Started

The main thing that you need to be able to start this project would be to have NPM installed and then to have the angular-cli installed.
You might want to look up what Oauth2 authorization is, but that would just be for understanding more than anything. Also, you will wanta good IDE to work with. I use and recommend jetbrains webstorm, but any IDE you like will do.

### Prerequisites

To install NPM go to https://www.npmjs.com/get-npm and it is a free javascript package manager that helps organize all of the packages that you will end up downloading. It it pretty easy to use and if you don't know how I would look up a quick tutorial on that as well.


### Installing


 You have to install the Angular-cli through npm. Write this in the command prompt after you have npm installed.
 
```
npm install angular-cli
```

After Angular is installed you will want to create a new angular project. Write this in the command prompt as well.

```
ng new PROJECT_NAME
```

Once you have an angular project up and running you should register yourself with the fhir sandbox DSTU2 server so you can test apps. Go here to make your account and login to the fhir sandbox that uses oauth2 and has the patient data we need to access.

```
https://sandbox.smarthealthit.org/#/start
```

After you login to the smart on fhir sandbox and click on the DSTU2 server, then go to registered apps and click on register manually. 

```
The app launch URI is the first page your website/app goes to and 
the redirect URI is the landing page after fhir authenticates the user
```
The launch URI should be localhost:4200/launch.html and the redirect URI should be localhost:4200/afterlaunch.html

After you have a registered app you should create a static launch.html page in your source folder

```
After you create the static html page in the source folder, then you should go to the angular cli json file and 
add "launch.html" in the assets bracket

```

Add both of those files to the angular assets so when you ng serve a local hosted webpage angular knows to load those files

```
After you create the static html page in the source folder, then you should go to the angular cli json file 
and add "afterlaunch.html" in the assets bracket

```

For the launch.html page you will want to include the launch code from the sample authorization page that shows an example

Make sure to change the client id and the redirect uri to what you need them to be for this code to work

```
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
        var redirectUri = launchUri.replace("launch.html","afterlaunch.html");
        
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
```

And for the afterlaunch.html page you will want to include this index code provided by the fhir website

```
<!DOCTYPE html>
<html>
  <head>
     <title>Simple Auth App</title>
     <script src="http://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
  </head>
  <body>
    <script>
        // get the URL parameters received from the authorization server
        var state = getUrlParameter("state");  // session key
        var code = getUrlParameter("code");    // authorization code
        
        // load the app parameters stored in the session
        var params = JSON.parse(sessionStorage[state]);  // load app session
        var tokenUri = params.tokenUri;
        var clientId = params.clientId;
        var secret = params.secret;
        var serviceUri = params.serviceUri;
        var redirectUri = params.redirectUri;
        
        // Prep the token exchange call parameters
        var data = {
            code: code,
            grant_type: 'authorization_code',
            redirect_uri: redirectUri
        };
        var options;
        if (!secret) {
            data['client_id'] = clientId;
        }
        options = {
            url: tokenUri,
            type: 'POST',
            data: data
        };
        if (secret) {
            options['headers'] = {'Authorization': 'Basic ' + btoa(clientId + ':' + secret)};
        }
        
        // obtain authorization token from the authorization service using the authorization code
        $.ajax(options).done(function(res){
            // should get back the access token and the patient ID
            var accessToken = res.access_token;
            var patientId = res.patient;
                    
            // and now we can use these to construct standard FHIR
            // REST calls to obtain patient resources with the
            // SMART on FHIR-specific authorization header...
            // Let's, for example, grab the patient resource and
            // print the patient name on the screen
            var url = serviceUri + "/Patient/" + patientId;
            $.ajax({
                url: url,
                type: "GET",
                dataType: "json",
                headers: {
                    "Authorization": "Bearer " + accessToken
                },
            }).done(function(pt){
                var name = pt.name[0].given.join(" ") +" "+ pt.name[0].family.join(" ");
                document.body.innerHTML += "<h3>Patient: " + name + "</h3>";
            });
        });
        
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
```

After you get this code into your angular app I would just make sure it works. Make sure you make the redirect URI, client ID, and scopes the same from the launch html and the launch information that the fhir sandbox needs to authenticate you correctly. This is what I have for my scopes so far. Change them in the html page and the launch information if needed.

```
Client Type: Public Client
Client Id: Your client Id
App Launch URI: http://localhost:4200/launch.html
App Redirect URI: http://localhost:4200/afterlaunch.html
Scopes: patient/*.read launch
```

The next step would be to create an angular service and component so you can start actually using angular. Write these two commands to create a component and a service in angular.

```
ng generate component afterlaunch
ng generate service authorization
```

After these two are created we will want to adapt our code that uses jquery requests with javascript in our afterlaunch.html file to making requests using the angular httpclient in typescript. 

Before we change the code and add it to the service component we need to add a couple things to the app.module.ts file in the app folder

```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';
import { AfterLaunchComponent } from './afterlaunch/afterlaunch.component';
import {RouterModule, Routes} from '@angular/router';
import {HttpClientModule} from '@angular/common/http';
import {FormsModule} from '@angular/forms';

const appRoutes: Routes = [
  {path: 'afterlaunch', component: AfterLaunchComponent},
];

@NgModule({
  declarations: [
    AppComponent,
    AfterLaunchComponent
  ]
  imports: [
    BrowserModule,
    HttpClientModule,
    FormsModule,
    RouterModule.forRoot(
      appRoutes,
      {enableTracing: true}
    )
  ],
  providers: [AfterLaunchComponent],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
We needed to add the import for the httpclient and router module and then create a route connecting the afterlaunch url to our landing component. Make sure the components and services matches the names you made them.

One last thing we need to do is to add the router outlet tag to the app.component.html file. We need to do this so that when we route our AfterLaunchComponent to the correct url that the right information will show to the user through the app component. 

Delete everything that was in the app component html file and replace it with this one tag. All of the routes should go up on this page when the url is correct for the route

```
<router-outlet></router-outlet>
```

Now here is how you change all of the variables from javascript to typescript and make all of the get and post requests through angular. 
Include this code in the new service that was just created.
```
import { Injectable } from '@angular/core';
import {ActivatedRoute} from '@angular/router';
import {isUndefined} from 'util';
import {HttpClient, HttpHeaders, HttpParams} from '@angular/common/http';
import {Observable} from 'rxjs/Observable';

@Injectable()
export class SmartAuthService {
  state: string;
  code: string;
  private sub: any;
  urlParams: any;

  // Session params
  tokenUri: string;
  clientId: string;
  secret: string;
  serviceUri: string;
  redirectUri: string;

  private accessToken: string;
  private patientId: string;

  constructor(private http: HttpClient) { }

  initialize(route: ActivatedRoute): void {
    this.sub = route.queryParams.subscribe(params => {
      this.urlParams = params;
    });

    // Get URL parameters from auth server
    this.state = this.getUrlParameter('state');
    this.code = this.getUrlParameter('code');

    // Load app parameters stored in session
    const sessionParams = JSON.parse(sessionStorage[this.state]);
    this.tokenUri = sessionParams.tokenUri;
    this.clientId = sessionParams.clientId;
    this.secret = sessionParams.secret;
    this.serviceUri = sessionParams.serviceUri;
    this.redirectUri = sessionParams.redirectUri;

    // Prep the token exchange call parameters
    const urlParams = new HttpParams()
      .set('code', this.code)
      .set('grant_type', 'authorization_code')
      .set('redirect_uri', this.redirectUri)
      .set('client_id', this.clientId);

    this.http.post(this.tokenUri, urlParams).subscribe(
      res => {
        console.log('received data from server');
        this.accessToken = res['access_token'];
        this.patientId = res['patient'];
      }
    );
  }
  getPatient() {
    if (!this.accessToken) {
      return Observable.throw('no access token');
    }

    return this.http.get(
      this.serviceUri + '/Patient/' + this.patientId, {
        headers: new HttpHeaders().set('Authorization', 'Bearer ' + this.accessToken)
      }
    );
  }

  private getUrlParameter(sParam: string): string {
    if (isUndefined(this.urlParams[sParam])) {
      console.log('parameter ' + sParam + ' does not exist');
      return '';
    }
    return this.urlParams[sParam];
  }

}
```

After you add this code to the authorization service then you have to make sure that the authorization works by using button press functions in the new component that you just made to make sure you can actually get to the right closed off information in the smart on fhir sandbox. Use this code to do so. 

This is what goes into the component.ts file.

```
import {Component, OnInit} from '@angular/core';
import {ActivatedRoute} from '@angular/router';
import {SmartAuthService} from '../authorization.service';

@Component({
  selector: 'app-landing',
  templateUrl: './afterlaunch.component.html',
  styleUrls: ['./afterlaunch.component.css'],
  providers: [SmartAuthService]
})
export class AfterLaunchComponent implements OnInit {
  ptName: string;

  constructor(private route: ActivatedRoute, private authService: SmartAuthService) {
  }

  ngOnInit() {
    this.authService.initialize(this.route);
  }

  getPatientName() {
    this.authService.getPatient().subscribe(data => {
      // get pt name and store it in a variable
      this.ptName = data['name'][0].given.join(' ') + ' ' + data['name'][0].family.join(' ');
    });
  }
}


```

And here is what you should put in the html part of the component

```
<h1>In afterlaunch Component:</h1>
<button (click)="getPatientName()"> Get Patient Name </button>
  <p> {{ptName}} </p>

  <p>End of landing Component</p>

```

After you include this button you should be able to ng serve and then launch the app from the fhir sandbox. After it is launched if you press the button then it should show the patient's name of who you chose to get the data for.

Also, there might be an error in the spec file for the authorzation service. If it needs to be fixed replace the file with this code.


```
import { TestBed, inject } from '@angular/core/testing';

import { SmartAuthService } from './authorization.service';

describe('AuthorizationService', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [SmartAuthService]
    });
  });

  it('should be created', inject([SmartAuthService], (service: SmartAuthService) => {
    expect(service).toBeTruthy();
  }));
});

```
Lastly, if there is an error about the afterlaunch component not being an exported file, then you should just replace the afterlaunch name with the red line under it with AfterLaunchComponent and do the same for all of the words that are covered in red. 


## Built With

* [Angular](https://angular.io/) - The web framework used


## Authors

* **Jason Judis**

