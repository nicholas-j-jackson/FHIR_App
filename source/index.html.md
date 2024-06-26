---
title: SMART on FHIR app tutorial

language_tabs:
  - code

search: true
---

# Introduction

This tutorial will walk you through creating an app in Cerner's SMART on FHIR ecosystem.

After completing this tutorial you will know how to:

* Create a basic SMART on FHIR app.
* Self register an app with Cerner.
* Run an app in Cerner's SMART on FHIR sandbox.
* Self register an app with SMART Health IT.
* Run an app in SMART Health IT Sandbox.
* Setup a standalone patient access app.

Note: This tutorial is in the process of being updated for the newer version of FHIR - [R4](http://fhir.cerner.com/millennium/r4/). Outside of this tutorial, the use of R4 over DSTU2 is encouraged.

# Prerequisites
  * A public [GitHub](http://www.github.com) account

# Project Setup

First, you'll want to access the tutorial at [smart-on-fhir-tutorial](https://github.com/cerner/smart-on-fhir-tutorial). Then, click on "Use this template" button to create a new repository with **smart-on-fhir-tutorial** as the repository name to your GitHub account. Keep the "Include all branches" unchecked, then click on "Create repository from template" button to complete. The branch you are going to work on is gh-pages.

The `smart-on-fhir-tutorial/example-smart-app` folder contains the example SMART app which you'll be using throughout this tutorial. Let's take a look at some of the notable files contained within:

**fhir-client-[version].js**

Located in the lib folder, this is a version of [fhir-client.js](https://github.com/smart-on-fhir/client-js) which is an open source library designed to assist with calling a FHIR API and handling the SMART on FHIR authorization workflow. This tutorial uses this library when walking you through building your first SMART app.

Additional documentation on fhir-client.js can be found [here](http://docs.smarthealthit.org/client-js/).

<aside class="notice">
This tutorial is designed to have a minimal footprint so we made the decision to directly include a version of fhir-client.js for simplicity. For your production applications we'd recommend pulling in the appropriate version of fhir-client.js using npm or some other package manager to easily keep your application up to date.
</aside>

**launch.html**

launch.html is the SMART app's initial entry point and in a real production environment, would be invoked by the application launching your SMART app (for instance, the EHR or patient portal). In the [SMART documentation](http://docs.smarthealthit.org/), this is your app's "launch URL". In this tutorial, this page will be invoked when you launch your app from Cerner's [code console](https://code.cerner.com/developer/smart-on-fhir/apps).

As the entry point into your SMART app, this page will kick-off the SMART authorization workflow.

**launch-patient.html**

Similar to the launch.html above, this file is the entry point when launching a standalone patient application. This file was created for convenience factor. In production, you may want to create a separate app for patient facing vs provider facing version of the app.  More info on this in [Standalone App Launch for Patient Access Workflow](#standalone-app-launch-for-patient-access-workflow) section.

**launch-smart-sandbox.html**

This is a clone of the launch.html above. This file was created for convenience factor to allow you to use the same app to configure it against the SMART Health IT Sandbox.  More info on this in [Run your app against SMART Health IT Sandbox](https://sandbox.smarthealthit.org) section.

**index.html**

This page will be invoked via redirect from the Authorization server at the conclusion of the SMART authorization workflow. When this page is invoked, your SMART app will have everything it needs to run and access the FHIR API.

The other content you see in the folder is the site for this tutorial. We used [Slate](https://github.com/lord/slate) to create the documentation for this tutorial.

# GitHub Pages

>index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv='X-UA-Compatible' content='IE=edge' />
    <meta http-equiv='Content-Type' content='text/html; charset=utf-8' />
    <title>[YOUR-USERNAME] Example-SMART-App</title>
    ...
```

> Go to your GitHub account, select Repositories tab and select smart-on-fhir-tutorial repo. Select Branch button and switch to gh-pages branch if it is not already selected. Directly edit `/example-smart-app/index.html` by clicking on the pencil icon.  Once done with the change, commit directly to gh-pages branch, this will ensure your changes are auto deployed by GitHub.

>The SMART app will be available at:

```
https://<gh-username>.github.io/smart-on-fhir-tutorial/example-smart-app/
```
>Health check

```
https://<gh-username>.github.io/smart-on-fhir-tutorial/example-smart-app/health
```

For the purposes of this tutorial we will be hosting our SMART app through [GitHub Pages](https://help.github.com/articles/what-is-github-pages). GitHub Pages is a convenient way to host static or client rendered web sites.

Setting up GitHub pages is easy, so easy in fact that it's already done for you. GitHub pages works by hosting content from a gh-pages branch. The gh-pages branch has already been created, however GitHub won't publish your site until you make a change to the gh-pages branch, so let's make a change. Modify the index.html page to include your GitHub user-name in the title, and commit directly to gh-pages branch.

Use GitHub UI to directly edit `index.html`. Simply switch the branch to gh-pages, navigate to `/example-smart-app/index.html` and click the pencil icon. Commit your changes to deploy.

Once the app has been redeployed go to ```https://<gh-username>.github.io/smart-on-fhir-tutorial/example-smart-app/health``` to ensure your app is available.

<aside class="notice">
GitHub Pages sites have a limit of 10 builds per hour, so if your page isn't updating, this could be the reason.
</aside>

# Registration
Now that we have a deployed SMART app, let's register it to access Cerner's FHIR resources. We have created a self registration console to allow any developer to be able to run a SMART app against our development environment. Navigate to our [code console](https://code.cerner.com/developer/smart-on-fhir/apps), if you don't have a Cerner Care Account, go ahead and sign up for one (it's free!). Once logged into the console, click on the "+ New App" button in the top right toolbar and fill in the following details:

Field | Description
--------- | -----------
App Name | ```My amazing SMART app``` Any name will do.
SMART Launch URI | ```https://<gh-username>.github.io/smart-on-fhir-tutorial/example-smart-app/launch.html```
Redirect URI | ```https://<gh-username>.github.io/smart-on-fhir-tutorial/example-smart-app/```
App Type | ```Provider``` Provider facing app
FHIR Spec | ```dstu2``` The latest spec version supported by Cerner.
Authorized | ```Yes``` Authorized App will go through secured OAuth 2 login.
Standard Scopes | These scopes are required to launch the SMART app.
User Scopes | None
Patient Scopes | Locate the ***Patient Scopes*** table and select the ***Patient*** read and ***Observation*** read scopes.

Specifying user scopes or patient scopes will result in a slightly different testing workflow. See this section: [Test your App](#test-your-app).
More information about Patient-specific scopes vs. User-level scopes can be found in this [spec](http://hl7.org/fhir/smart-app-launch/scopes-and-launch-context/index.html#scopes-for-requesting-clinical-data).

Click "Register" to complete the process. This will add the app to your account and create a client id for app authorization.

The new OAuth 2 client id will be displayed in a banner at the top of the page and can be viewed at any time by clicking on the application icon to view more details.

# App Launch

<aside class="notice">
After initially registering your SMART app, it can take up to 10 minutes for your app details to propogate throughout our sandbox. So, please wait 10 minutes before trying to launch your app. Don't fret, we're working on fixing this!
</aside>

## Provider App

We have now created our own SMART app and registered that app with Cerner to access the FHIR resources. Before we continue on with the next steps, let's take a moment to talk about the flow of a SMART app launch.

The SMART app launch flow begins with the EHR. Through some method, a user has indicated that they wish to launch a smart application. The EHR redirects to the SMART ```Launch URI``` that was registered above.

In this example ```Launch URI``` is launch.html. launch.html redirects to the FHIR authorization server which in-turn redirects to the ```Redirect URI```, index.html, upon a successful authentication.

Post-authentication, index.html exchanges the returned authorization token for an access token and is then able to request resources from the FHIR server. Let's take a deeper look at launch.html and get it ready for authentication. For more information about the SMART app launching vist the [SMART Health IT site](http://docs.smarthealthit.org/authorization/).

![alt text](ehr_launch_seq.png "High Level EHR App Launch Flow")

EHR App Launch Flow - Full size image [here](images/ehr_launch_seq.png)

## Patient App

Unlike the EHR app launch flow above, a standalone app does not need to be launched by an EHR or a patient portal. Cerner currently supports the special "launch/patient" scope that can be used during a standalone launch to request that the user must select a patient during authorization. However, this scope is currently only supported for patient launches. If you do a standalone launch for provider, your application will be responsible for presenting a patient search in order for a provider to select a patient's chart. You can learn more about the standalone launch at [SMART Health IT site](http://docs.smarthealthit.org/authorization/).

There are a few minor differences:

* Patients can launch any standalone app
* App provides the iss param of the FHIR server
* App can request launch/patient scope to obtain a patient in context, if the application is using any SMART patient/* launch scopes

More information on how to configure a patient standalone app can be found in [Standalone App Launch for Patient Access Workflow](#standalone-app-launch-for-patient-access-workflow) section below.

![alt text](patient_launch_seq.png "High Level Patient App Launch Flow")

Patient App Launch Flow - Full size image [here](images/patient_launch_seq.png)

# Request Authorization

> launch.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>Example-SMART-App</title>
  </head>
  Loading...
  <body>
    <script src='./lib/fhir-client-v0.1.11.js'></script>
    <!-- Prevent session bleed caused by single threaded embedded browser and sessionStorage API -->
    <!-- https://github.com/cerner/fhir-client-cerner-additions -->
    <script src='./lib/fhir-client-cerner-additions-1.0.0.js'></script>
    <script>
      FHIR.oauth2.authorize({
        'client_id': '<enter your client id here>',
        'scope':  'patient/Patient.read patient/Observation.read launch online_access openid profile'
      });
    </script>
  </body>
</html>
```

> Make sure to replace CLIENT_ID with the client id provided in code console and redeploy your site.

The responsibility of launch.html is to redirect to the appropriate FHIR authorization server. As you can see in the code, fhir-client makes our job pretty easy. All we have to do is call ```FHIR.oauth2.authorize``` and supply the client_id generated by the code console during registration and the scopes we registered.

The client_id is found in the app details page that can be accessed by clicking on the application icon in the [code console](https://code.cerner.com/developer/smart-on-fhir/apps). Copy the client_id into the authorize call in launch.html, commit the changes back to your repo and redeploy your site.

For the purposed of this tutorial you don't need to modify the scopes. This list should match the scopes that you registered the application with.

Below is some additional information about the scopes we've selected for our app.

Scope | Grants
--------- | -----------
patient/Patient.read | Permission to read Patient resource for the current patient.
patient/Observation.read | Permission to read Observation resource for the current patient.
openid, profile | Permission to retrieve information about the current logged-in user. Required for EHR launch.
launch | Permission to obtain launch context when app is launched from an EHR. Required for EHR launch.
launch/patient | Permission to have a patient be selected when performing a standalone launch. Currently supported only for Patient standalone launch. Required for a standalone launch if the application attempts to use patient/* type SMART scopes. For example, if an app uses only user/Patient.read or user/Observation.read scopes it wouldn't use launch/patient scope. See this section: [Standalone App Launch for Patient Access Workflow](#standalone-app-launch-for-patient-access-workflow).
online_access | Request a refresh_token that can be used to obtain a new access token to replace an expired one, and that will be usable for as long as the end-user remains online. Required for EHR launch.

For our app we will use Patient.read, Observation.read.
We will always include launch, online_access, openid & profile scopes to our app.

<aside class="notice">
Cerner does not allow use of wildcards(*). So instead of patient/*.read you will need to specify a particular scope of resource you will be using. Something like patient/Patient.read, patient/Observation.read etc. For the list of resources, visit <a href='http://fhir.cerner.com'>http://fhir.cerner.com/</a>.
</aside>

So just what exactly is the ```FHIR.oauth2.authorize``` method doing?

Through an EHR launch, launch.html will be supplied with two query params ```iss``` and ```launch```

```iss``` is the EHR's FHIR end point and ```launch``` is an identifier that will be passed along to the authorization server.

```FHIR.oauth2.authorize``` queries the FHIR endpoint to find the URI for authorization.
It then simply redirects to that endpoint, filling out the required API which includes the supplied client_id, scopes and the launch parameter passed in from the EHR. (There are a few more params that can be read about [here](http://docs.smarthealthit.org/authorization/)). Additionally the function generates an appropriate ```state``` parameter that will then be checked after redirecting to the index page.

Following the ```FHIR.oauth2.authorize```, the app will redirect to the authorization server, which, on a successful authorization, will redirect back to the ```Redirect URI```, in this case, index.html

<aside class="notice">
The OAuth 2 client id is an identifier, not a secret. As such, it does not need to be hidden. It's used in conjunction with the other information provided through app registration, such as the redirect URI, to launch your application. If another app has access to your OAuth 2 client id they will not be able to masquerade as your application.
</aside>

# Access Token Retrieval

>index.html

```html
...
<script src='./lib/es6-shim-0.35.1.min.js'></script>
<script src='./src/js/example-smart-app.js'></script>
<script src='./lib/fhir-client-v0.1.11.js'></script>

<!-- Prevent session bleed caused by single threaded embedded browser and sessionStorage API -->
<!-- https://github.com/cerner/fhir-client-cerner-additions -->
<script src='./lib/fhir-client-cerner-additions-1.0.0.js'></script>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
<script>
  extractData().then(
    //Display Patient Demographics and Observations if extractData was success
    function(p) {
      drawVisualization(p);
    },

    //Display 'Failed to call FHIR Service' if extractData failed
    function() {
      $('#errors').html('<p> Failed to call FHIR Service </p>');
    }
  );
</script>
...
```

> example-smart-app.js - extractData

```javascript
...
window.extractData = function() {
    var ret = $.Deferred();
    ...
    ...
    FHIR.oauth2.ready(onReady, onError);
    return ret.promise();
  };
```

Now that the app has successfully been authenticated, it's time to call a FHIR resource, but first we need to obtain an OAuth2 access token. We have an authorization code that was passed as a query param to the redirect URI (index.html) by the authorization server. The authorization code is exchanged for an access token through POST to the authorization server. Again, fhir-client.js makes this easy for us.

The `index.html` file includes a script which calls into the ```extractData``` function in `example-smart-app.js`.

```extractData``` uses the ```FHIR.oauth2.ready()``` function to exchange the authorization code for the access token and stores it in session storage for later use.

# Access FHIR Resource
> example-smart-app.js - onReady

```javascript
...
function onReady(smart)  {
  if (smart.hasOwnProperty('patient')) {
    var patient = smart.patient;
    var pt = patient.read();
    var obv = smart.patient.api.fetchAll({
                  type: 'Observation',
                  query: {
                    code: {
                      $or: ['http://loinc.org|8302-2', 'http://loinc.org|8462-4',
                            'http://loinc.org|8480-6', 'http://loinc.org|2085-9',
                            'http://loinc.org|2089-1', 'http://loinc.org|55284-4']
                          }
                         }
                });

    $.when(pt, obv).fail(onError);

    $.when(pt, obv).done(function(patient, obv) {
      var byCodes = smart.byCodes(obv, 'code');
      var gender = patient.gender;
      var dob = new Date(patient.birthDate);
      var day = dob.getDate();
      var monthIndex = dob.getMonth() + 1;
      var year = dob.getFullYear();

      var dobStr = monthIndex + '/' + day + '/' + year;
      var fname = '';
      var lname = '';

      if(typeof patient.name[0] !== 'undefined') {
        fname = patient.name[0].given.join(' ');
        lname = patient.name[0].family.join(' ');
      }

      var height = byCodes('8302-2');
      var systolicbp = getBloodPressureValue(byCodes('55284-4'),'8480-6');
      var diastolicbp = getBloodPressureValue(byCodes('55284-4'),'8462-4');
      var hdl = byCodes('2085-9');
      var ldl = byCodes('2089-1');

      var p = defaultPatient();
      p.birthdate = dobStr;
      p.gender = gender;
      p.fname = fname;
      p.lname = lname;
      p.age = parseInt(calculateAge(dob));

      if(typeof height[0] != 'undefined' && typeof height[0].valueQuantity.value != 'undefined' && typeof height[0].valueQuantity.unit != 'undefined') {
        p.height = height[0].valueQuantity.value + ' ' + height[0].valueQuantity.unit;
      }

      if(typeof systolicbp != 'undefined')  {
        p.systolicbp = systolicbp;
      }

      if(typeof diastolicbp != 'undefined') {
        p.diastolicbp = diastolicbp;
      }

      if(typeof hdl[0] != 'undefined' && typeof hdl[0].valueQuantity.value != 'undefined' && typeof hdl[0].valueQuantity.unit != 'undefined') {
        p.hdl = hdl[0].valueQuantity.value + ' ' + hdl[0].valueQuantity.unit;
      }

      if(typeof ldl[0] != 'undefined' && typeof ldl[0].valueQuantity.value != 'undefined' && typeof ldl[0].valueQuantity.unit != 'undefined') {
        p.ldl = ldl[0].valueQuantity.value + ' ' + ldl[0].valueQuantity.unit;
      }
      ret.resolve(p);
    });
  } else {
    onError();
  }
}
...
```
With access token in hand we're ready to request a FHIR resource and again, we will be using fhir-client-[version].js.

For the purposes of this tutorial we'll be retrieving basic information about the patient and a couple of basic observations to display.

The fhir-client-[version].js library defines several useful API's we can use to retrieve this information.

* ```smart.patient.read()```
  * This will return the context for the patient the app was launched for.
* ```smart.patient.api```
  * ```fetchAll()```
    * This will use the [fhir.js](https://github.com/FHIR/fhir.js) API to retrieve a complete set of resources for the patient in context.

Both of these functions will return a jQuery deferred object which we unpack on success.

Unpacking is fairly straight forward. We're taking the response from the patient and observation resources and placing it into a "patient" data structure.

The last function from fhir-client-[version].js is the ```byCodes``` utility function that returns a function to search a given resource for specific codes returned from that response.

The fhir-client-[version].js library defines several more API's that will come in handy while developing smart app. Read about them [here](http://docs.smarthealthit.org/client-js/).

# Displaying the Resource

>index.html

```html
...
<h2>SMART on FHIR Starter App</h2>
<div id='errors'>
</div>
<div id="loading">Loading...</div>
<div id='holder' >
  <h2>Patient Resource</h2>
  <table>
    <tr>
      <th>First Name:</th>
      <td id='fname'></td>
    </tr>
    <tr>
      <th>Last Name:</th>
      <td id='lname'></td>
    </tr>
    <tr>
      <th>Gender:</th>
      <td id='gender'></td>
    </tr>
    <tr>
      <th>Date of Birth:</th>
      <td id='birthdate'></td>
    </tr>
    <tr>
      <th>Age:</th>
      <td id='age'></td>
    </tr>
  </table>
  <h2>Observation Resource</h2>
  <table>
    <tr>
      <th>Height:</th>
      <td id='height'></td>
    </tr>
    <tr>
      <th>Systolic Blood Pressure:</th>
      <td id='systolicbp'></td>
    </tr>
    <tr>
      <th>Diastolic Blood Pressure:</th>
      <td id='diastolicbp'></td>
    </tr>
    <tr>
      <th>LDL:</th>
      <td id='ldl'></td>
    </tr>
    <tr>
      <th>HDL:</th>
      <td id='hdl'></td>
    </tr>
  </table>
</div>
...
```

>example-smart-app.js - drawVisualization

```javascript
...
window.drawVisualization = function(p) {
  $('#holder').show();
  $('#loading').hide();
  $('#fname').html(p.fname);
  $('#lname').html(p.lname);
  $('#gender').html(p.gender);
  $('#birthdate').html(p.birthdate);
  $('#age').html(p.age);
  $('#height').html(p.height);
  $('#systolicbp').html(p.systolicbp);
  $('#diastolicbp').html(p.diastolicbp);
  $('#ldl').html(p.ldl);
  $('#hdl').html(p.hdl);
};
...
```

The last remaining task for our application is displaying the resource information we've retrieved. In `index.html` we define a table with several id place holders. On a success from ```extractData``` we'll call ```drawVisualization``` which will show the table div as well as filling out the relevant sections.

# Test your App
> To re-deploy the GitHub Pages site, commit your changes and make sure your gh-pages branch is up to date.

Now that we have a snazzy SMART app, it's time to test it.

* First, log back into the [code console](https://code.cerner.com/developer/smart-on-fhir/apps) and click on the app you've registered (My amazing SMART app).
* To launch your app through the code console click the "Begin Testing" button.
  * If the app has a user scope, the console will ask if the app you're launching requires a patient in context. Because our app only has patient scopes, the question will be skipped and you just need to choose a patient to continue.
  * You can choose whether to display a demographics banner for the patient. This need_patient_banner flag will be part of the launch parameters. However, this tutorial app currently does not utilize this flag when displaying the result.
* Click "Next" to open a "Ready to launch" popup. <span style="color: red;">Take a note of the testing username and password</span>, you'll need this credential when prompted.
* Finally, click "Launch" and the console will redirect to your application.

# MPages® Integration

MPages® is a Web-based platform that enables clients to create customized views of Cerner Millennium® data at the organizer or chart level from within Cerner PowerChart®, FirstNet®, INet® and SurgiNet®.

There are a few different files and HTML tags you need to add to each view within your application to securely embed the SMART App within
an MPage® view.

>index.html - launch.html - health.html

```html
<html lang="en"  hidden>
  <head>
    <meta http-equiv='X-UA-Compatible' content='IE=edge' />
    ...
    <link rel='stylesheet' type='text/css' href='./lib/css/cerner-smart-embeddable-lib-[version].min.css'>
    ...
```

```html
  <body>
    ...
    <script src='https://cdnjs.cloudflare.com/ajax/libs/babel-polyfill/6.26.0/polyfill.min.js'></script>
    <script src='./lib/js/cerner-smart-embeddable-lib-[version].min.js'></script>
    ...
```


- Include the `hidden` attribute to the `html` tag.
- Set a `meta` tag at the top of the `head` tag to display Internet Explorer content in the highest compatibility mode.
- Include the `babel-polyfill` module into the project to support ES2015+ JavaScript standard methods and Objects that this project (and included libraries) may use.
- Include the `cerner-smart-embeddable-lib` files that utilize the [XFC](https://github.com/cerner/xfc) (Cross-Frame-Container) library to prevent possible [Clickjacking
attacks](https://www.owasp.org/index.php/Clickjacking) in any browser. These files can be pulled in from the [Cerner SMART Embeddable Library](https://github.com/cerner/cerner-smart-embeddable-lib). See project
description for how to properly size the application, and note additional considerations around conflicting HTTP headers.

Note: The steps above only ensure that the application will meet certain prerequisites to securely embed a SMART app within an MPage® view.
Once these relevant files and HTML tags are included inside each view of the app, then the application should be ready for SMART in MPage® integration.

# Run your app against SMART Health IT

One of the reasons why SMART on FHIR is awesome is because of the interoperability factor!  If an EHR follows the SMART and FHIR specifications, your application will work with that EHR's SMART on FHIR implemenmtation.  Let's see if the app that you've built will work with [SMART Launcher](https://launch.smarthealthit.org).  The following steps will walk you through setting up your app at SMART Health IT site.

* Go to [https://launch.smarthealthit.org](https://launch.smarthealthit.org)
* In the **App Launch Options** select **Launch Type** as 'Provider EHR Launch' with 'Simulate launch within the EHR user interface' box checked. This is the default.
* Under **FHIR Version**, select 'R2 (DSTU2)'
* In the **Patient(s)** section, click the drop down button to open a list of patients in a new pop up window. Select any patient and click 'OK'. This should populate the patient ID in the field.
* In the **Provider(s)** section, click the drop down button to open a list of providers. Select any provider to populate the Provider ID in the field.
* For the purpose of this tutorial we will leave the **Advanced** options as is.
* In the **Launch** section, use the following value but replace `<gh-username>` with your GitHub username for **App Launch URL**:
https://`<gh-username>`.github.io/smart-on-fhir-tutorial/example-smart-app/launch-smart-sandbox.html
* Note: Currently, the [SMART App Launcher](https://launch.smarthealthit.org) does not check/validate the `client_id` field. So, any value for `client_id` is fine. If/when this changes, or when working with other authorization servers, please update the `client_id` field in `launch-smart-sandbox.html` file.
* Now launch the app by clicking on the green 'Launch App!' button to see your app opened in the simulated EHR with the patient data.


> launch-smart-sandbox.html

```
...
<!-- Currently, the SMART App Launcher (https://launch.smarthealthit.org) does not check/validate the client_id field.
        If/when this changes, or when working with other authorization servers, please update the client_id field here. -->
<script>
  FHIR.oauth2.authorize({
    'client_id': 'YOUR-SMART-HEALTH-IT-CLIENT-ID-HERE',
    'scope':  'patient/Patient.read patient/Observation.read launch online_access openid profile'
  });
</script>
...
```

# Standalone App Launch for Patient Access Workflow

SMART supports EHR launch and standalone launch. With a standalone launch, SMART provides a scope that allows the application to request that the user chooses a patient during authorization (launch/patient). Cerner currently only supports the launch/patient scope for patient access workflow, a practitioner access application would need to provide the patient search itself during a standalone launch. The standalone application does not need to be launched by an EHR or a patient portal. The app can launch and access FHIR data on its own, provided the app is authorized and given or configured with the iss (FHIR server root) URL.

You can find out more about standalone launch on the [SMART Health IT site](http://docs.smarthealthit.org/authorization/) under the "Standalone launch sequence" header.

Since patient access is specifically for patient workflow, we need to create another app in [code console](https://code.cerner.com/developer/smart-on-fhir/apps).  Below, you can follow the instruction to perform this registration.

## Registration
Navigate to our [code console](https://code.cerner.com/developer/smart-on-fhir/apps). Once logged in, click on the "+ New App" button in the top right toolbar and fill in the following details:

Field | Description
--------- | -----------
App Name | ```My amazing SMART Patient App``` Any name will do.
SMART Launch URI | Leave this field blank since this is a standalone patient app.
Redirect URI | ```https://<gh-username>.github.io/smart-on-fhir-tutorial/example-smart-app/```
App Type | ```Patient``` Patient facing app
FHIR Spec | ```dstu2``` The latest spec version supported by Cerner.
Authorized | ```Yes``` Authorized App will go through secured OAuth 2 login.
Standard Scopes | These scopes are required to launch the SMART patient app.
User Scopes | None
Patient Scopes | Locate the ***Patient Scopes*** table and select the ***Patient*** read and ***Observation*** read scopes.

Click "Register" to complete the process. This will add the app to your account and create a client id for app authorization.

The new OAuth 2 client id will be displayed in a banner at the top of the page and can be viewed at any time by clicking on the application icon to view more details.

## Request Authorization

The `launch-patient.html` file had been created for you already.  You'll need to update the client id with the new one you've just gotten. This file shows what your app will use to request authorization with the Authorization server.

> launch-patient.html

```html
  <script>
    FHIR.oauth2.authorize({
      'client_id': '<enter your client id here>',
      'scope':  'patient/Patient.read patient/Observation.read launch/patient online_access openid profile'
    });
  </script>
```

> Make sure to replace CLIENT_ID with the new client id provided in code console. Notice that `launch/patient` is used here instead of `launch`.  This is because during our standalone launch we want the authorization server to prompt the user to select a patient, in order to be able to use the patient/Patient.read and patient/Observation.read scopes above, which require a patient in context.

Finally, save `launch-patient.html` file in `gh-pages` branch.

## Launch a Standalone App

> Cerner's Sandbox Patient Access Endpoint:

```
https://fhir-myrecord.cerner.com/dstu2/ec2458f2-1e24-41c8-b71b-0e701af7583d
```

The `iss` value in the query parameter represents the URL for our Sandbox Patient Access endpoint.  This value tells the app where to look for the `metadata` endpoint, which contains the authorization endpoints that the app needs to call.

> Launch URL:

```
  https://<gh-username>.github.io/smart-on-fhir-tutorial/example-smart-app/launch-patient.html?iss=https://fhir-myrecord.cerner.com/dstu2/ec2458f2-1e24-41c8-b71b-0e701af7583d
```

Since this app is a standalone app, it does not need to be launched by the EHR or patient portal (code console).

Replace `<gh-username>` with your GitHub's username. Then, enter the URL above into the browser, the browser should redirect to the login page. You can use patients' credentials listed in the first post in this [discussion](https://groups.google.com/forum/#!topic/cerner-fhir-developers/edPUbVPIag0).

Once you authenticated and authorized the app, the app should load with Patient and Observation data.

# Next Steps
Through this tutorial we have:

* Created a basic SMART on FHIR app.
* Registered that app with Cerner.
* Run the app in Cerner's SMART on FHIR sandbox.
* Registered that app with SMART Health IT Sandbox.
* Run the app in SMART Health IT Sandbox.
* Setup a standalone patient access app.

We've created a very basic application that meets the base requirements of being a SMART app. This application would require a fair amount of polish before being ready to be deployed in a production environment. A couple of next steps you could look at are:

* Try calling another resource.
* Write unit tests for the application.
* Pull in the appropriate version of fhir-client.js through a package manager like webpack.
* Localize and Internationalize your application.

We're excited to see what you'll build next!
