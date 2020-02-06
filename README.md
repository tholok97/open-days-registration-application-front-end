# General description

This repository has been created during the open sourcing process of the 2019 Open Days ticket reservation system. The repository contains two Front-end projects written in [Angular](https://angular.io/) using [Angular Material design](https://material.angular.io/) and scripts for building a portable [Docker](https://www.docker.com/) container, which hosts the two applications using Apache httpd. 

You can find more information about the ticket reservation system in this [article](https://db-blog.web.cern.ch/blog/viktor-kozlovszky/2019-10-open-days-reservation-systems-high-level-overview-2019).

The content is orginized into different folders. For each folder in this README you will find a section which describes more detailed about its content. Here you can find a quick overview about the folder structure:
* **application**: The ticket reservation application which served as the user interface for reserving the tickets. The application works by making API calls against the back-end server (business layer). Please note that in order to use or test the functionalities of the application you need to have a back-end server up and running which recieves and processes the API calls. The back-end project for can be found in this [repository](). For more detailed information check out the [ticket reservation application section](#Ticket-reservation-application).
* **application-welcome**: This is a language selector page, it was sitting at the root level of the URL. Based on the language selection choice it forwards the user to the selected language path forexample: "/en/index.html" or "/fr/index.html". The need for this helper application was to select the language preference as first step and to have the same header styling for the application. For more detailed information check out the [langauge preference selector application section](#Langauge-preference-selector-application).
* **containerize**: This folder contains the helper files to build a container which hosts the two applications. For more detailed information check out the [container approach section](#Container-approach).

# Collaborators

The final application is the collaborated work of the following people:
* [] - ()
* [] - ()
* [Vik](https://github.com/kviktorman)
* [Thomas Løkkeborg](https://github.com/tholok97)

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

**containerize folder**

This folder contains helper files for the image build. In case you want to use the applications not as container you have to copy these files to the right places. 

```
>COPY containerize/scripts /opt/scripts
>COPY containerize/configuration/apache /opt/apache
```

- containerize folder content 
* * apache folder 
Contains the apache configurations for url rewrite more information [here](#URL-redirect)

* * missing folder

Note that the CERN specific logos and backgrounds are not included in the repository. These were originally under the "containerize/configuration/web-root/images". In case you want to build your container you need to create this folder and add images with the following names:
* opendays_background.png => background image of the application
* Site_web_logo.png =>  banner top left CERN Opendays reservation system logo 
* Site_web_opendays.png =>  banner top right CERN officiel site text logo

These files were hosted on the same level (top) as the language selector page. In case you don't provide these files you will see broken image icon in the browser once you load the application.

- scripts

Start up script which starts when container starts and runs the httpd as a foreground process.
