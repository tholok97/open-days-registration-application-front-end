# General description

This repository has been created during the open sourcing process of the 2019 Open Days ticket reservation system. The repository contains two Front-end projects written in [Angular](https://angular.io/) using [Angular Material design](https://material.angular.io/) and scripts for building a portable [Docker](https://www.docker.com/) container, which hosts the two applications using Apache httpd. 

You can find more information about the ticket reservation system in this [article](https://db-blog.web.cern.ch/blog/viktor-kozlovszky/2019-10-open-days-reservation-systems-high-level-overview-2019).

The content is orginized into different folders. For each folder in this README you will find a section which describes more detailed about its content. Here you can find a quick overview about the folder structure:
* **application**: The ticket reservation application which served as the user interface for reserving the tickets. The application works by making API calls against the back-end server (business layer). Please note that in order to use or test the functionalities of the application you need to have a back-end server up and running which recieves and processes the API calls. The back-end project for can be found in this [repository](). For more detailed information check out the [ticket reservation application section](#Ticket-reservation-application).
* **application-welcome**: This is a language selector page, it was sitting at the root level of the URL. Based on the language selection choice it forwards the user to the selected language path forexample: "/en/index.html" or "/fr/index.html". The need for this helper application was to select the language preference as first step and to have the same header styling for the application. For more detailed information check out the [langauge preference selector application section](#Langauge-preference-selector-application).
* **containerize**: This folder contains the helper files to build a container which hosts the two applications. For more detailed information check out the [container approach section](#Container-approach).

# Collaborators

The final application is the collaborated work of the following people:
* [Nuno Guilherme Matos de Barros](https://github.com/ngmatos)
* [Thomas Løkkeborg](https://github.com/tholok97)
* [Vik](https://github.com/kviktorman)

# Container approach

**environment variables**

In the docker file you need to set the following variables to their default values. Note that if you use Kubernetes for hosting the container you can override the default values in the deployment templates.

| variable name | description |
| --- | --- |
| REGISTRATION_API_URL | The URL of the back-end application server. |
| RECAPTCHA_SITE_KEY | site key for google re-captcha. In case you don't want to use it set to any string value and set the RECAPTCHA_ENABLED to false. |
| RECAPTCHA_ENABLED | true or false value  |
| PIWIK_SITE_ID | site id provided by PIWIK. This can be set to any string in case you don't want to use it. If you don't set there will be an error message in the console of the browser, but the application will be fully functional. |
| PIWIK_DOMAIN | the url domain of your application for PIWIK. This can be set to any string in case you don't want to use it. If you don't set there will be an error message in the console of the browser, but the application will be fully functional. |

**URL redirect**

Each of the two applications are designed as [Single Page Applications](https://en.wikipedia.org/wiki/Single-page_application) (SPAs). This means that there is only one html file + several js and css files for the applications. To route the non existing locations back to the same index.html file we used ".htaccess" files. There in total 3 of these files one for the language selector application and one for the english and one for french reservation application. The english end french files have "_en" and "_fr" endings, these are removed during the Docker container build process.

Note that in the container we use the apache URL rewrite. That needs to be installed first on the machine if you tend not to use not the container. 

**Containerize folder**

This folder contains helper files for the image build. In case you want to use the applications not as container you have to copy these files to the right places. 

```
>COPY containerize/scripts /opt/scripts
>COPY containerize/configuration/apache /opt/apache
```

- containerize folder content 
  * "apache" folder, contains the apache configurations for url rewrite more information [here](#URL-redirect)

  * missing folder

Note that the CERN specific logos and backgrounds are not included in the repository. These were originally under the "containerize/configuration/web-root/images". In case you want to build your container you need to create this folder and add images with the following names:
* opendays_background.png => background image of the application
* Site_web_logo.png =>  banner top left CERN Opendays reservation system logo 
* Site_web_opendays.png =>  banner top right CERN officiel site text logo

These files were hosted on the same level (top) as the language selector page. In case you don't provide these files you will see broken image icon in the browser once you load the application.

- scripts

Start up script which starts when container starts and runs the httpd as a foreground process.

# Use the container

To run the image you need to have [Docker installed](https://docs.docker.com/install/).

Once your environemnt is ready you build first the container. You navigate to the same level as the Dockerfile and run

```
>docker build -t reservation-system-front-end -f ./Dockerfile .
```

To run the image you need to forward the port 

```
>docker run -it -p 80:80 --rm --name reservation-front-end reservation-system-front-end
```

You access your image via http://localhost

# Langauge preference selector application

For the language selector page first we went for a non-Angular solution, but during the testing we noticed that certain browsers on phones zoom the top banner for the angular applications. In order to have the same banner size we decided to create an individual application. This application you can find under the application-welcome folder. The built application was hosted at the root level of the web server.

* using development environment 
The application is written in Angular. For investigation and further development process it is recommended to install the node package manager and the angular-cli.
More information how to set up your environment can find [here](https://angular.io/guide/setup-local).

* background URL replacement
In case you want to have an image as a background. This is not necessary for playing with the application.
Replace line 18 in "application-welcome/src/app/styles.scss" the final location of the background URL. 
```
  background: url(http://localhost/opendays_background.png) no-repeat center top/cover;
```



After the installation go to the folder application-welcome folder 

Install the dependency packages with the following command
```
>npm install
```

You can run the application with the following command. 

```
>ng serve
```

You access the application from the browser at the following URL: http://localhost:4200

* using image

If you want only to see the front-end application in action you use the built container image. You can read more about this approach [here](#Use-the-container).
