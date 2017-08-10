[![Join the chat at https://gitter.im/opsani/skopos](https://badges.gitter.im/opsani/skopos.svg)](https://gitter.im/opsani/skopos?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Sample Skopos application
===========================

This is a sample containerized application that can be deployed and upgraded with [Skopos](http://opsani.com/skopos/).

The application in question exposes two web interfaces - one that allows votes to be cast and one that shows results. The example below can be used as a guide on how to deploy, upgrade (to a version with modified UI) and tear-down this sample application.

![skopos sample app](skopos-sample-app.png)

### Download/Run Skopos image

```
docker run -d -p 8100:8100 --restart=unless-stopped --name skopos -v /var/run/docker.sock:/var/run/docker.sock opsani/skopos:edge
```

### Open Your Browser
Open your browser to ```http://localhost:8100``` Note: replace localhost with the actual host or IP address where Skopos runs.

### Clone the repository
We will need the application model, environment file and same sample scripts which we are using in our model in order to hook up into various stages of the deploy.

![Skopos Main Screen](http://opsani.com/wp-content/uploads/2017/08/Discover1.png)

Click the ```Use Our Demo App``` option.
Name your demo app and click

![Skopos Main Screen]()

### Start Skopos engine

Start the Skopos container, passing it the docker socket (so it can manage containers) and the user scripts from this repository. These are sample scripts that illustrate how to hook external systems (i.e. monitoring, release/bug tracking, etc.)  systems into your deploy process.

```
cd skopos-sample-app
docker run -d -p 8090:80 --restart=unless-stopped --name skopos   \
    -v /var/run/docker.sock:/var/run/docker.sock                  \
    -v $(pwd)/scripts:/skopos/user/bin/         \
    datagridsys/skopos:beta
```

### Load model for initial deploy

```
~/bin/sks-ctl load -bind my-ip-or-host:8090 -project skopos-sample -env env.yaml model-v1.yaml
```
Note: replace `my-ip-or-host` with the actual host or IP address where Skopos runs.


### Open Skopos GUI
It is available on port 8090 on the host where you are running the container. Review the model - it shows in blue the components that will change during the deploy.

You can switch the to plan (icon in upper right corner) to view the generated plan for this particular deploy. The plan shows the top level of steps to be run (one for each component plus pre- and post- flight steps). The plan would take into consideration any dependencies between components and upgrade them in the correct order. Each of the top level steps can be expanded to view the set of steps that will be performed for each component. The outcome of each step can trigger either the next step (on success) or a rollback to the previous version (on failure).


Some of the steps are inserted based on [lifecycle](http://skopos-beta.datagridsys.com/LIFECYCLE-REF/#application-lifecycle) events that were defined in the model file - this allows us to insert our own scripts to be executed at certain places in the plan:

![plan](plan.png)


### Run deploy
Either from the GUI or from CLI. The initial deploy may take a few minutes since container images will need to be downloaded.

```
~/bin/sks-ctl start -bind my-ip-or-host:8090
```

Note: replace `my-ip-or-host` with the actual host or IP address where Skopos runs.

After the deploy, the web interface exposed by the sample application would be available at:

* Vote: http://my-ip-or-host:8880/
* Result: http://my-ip-or-host:8881/

### Upgrade to a new version
This repository contains a second model, where the versions of two of the components - result and vote - are updated to 2.0. You can load the new model with the command below. Skopos would generate a plan for getting from the current state (v1.0) to the desired state as described in the model (v2.0 of vote and result components).


```
~/bin/sks-ctl load -bind my-ip-or-host:8090 -project skopos-sample -env env.yaml model-v2.yaml
```

Note: replace `my-ip-or-host` with the actual host or IP address where Skopos runs.

Review the new plan in UI. Notice how, unlike the initial deploy, it only changes two components and instead of a deploy it does a rolling upgrade, making sure each component stays responsive during the operation.

To start the deploy, use the GUI or the CLI as described above. If you open the result and vote web interfaces during the upgrade you will see how some requests are served by the old version and some are served by the new version, but the application is never inaccessible.


### Tear down the application
If you want to remove all containers for our sample application, run the following command. This example loads and starts the deploy in a single command. Progress can be still viewed in the Skopos UI (or over the API).

```
~/bin/sks-ctl run -bind my-ip-or-host:8090 -mode teardown -project skopos-sample -env env.yaml model-v2.yaml
```

Note: replace `my-ip-or-host` with the actual host or IP address where Skopos runs.
