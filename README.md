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

After you login to the smart on fhir sandbox go to  registered apps and click on register manually

'''
The app launch URI is the first page your website/app goes to and the redirect URI is the landing page after fhir authenticates the user
'''

After you have a registered app create a static javascript launch.html page in your app folder

'''
go  to 
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
