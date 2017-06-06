+++
date = "2015-05-26T16:26:45+02:00"
draft = false
title = "GO applications and Cloud Foundry"
+++
Cloud Foundry is one of many options to host your applications. It is a PaaS originally developed by VMware, now governed by <a href="http://cloudfoundry.org" target="_blank">Cloud Foundry Foundation</a>. Not going much into details about where, how and what as those information is publicly available I'll focus here on how to host your GO applications on any Cloud Foundry installation including <a href="http://pivotal.io/platform-as-a-service/pivotal-cloud-foundry" target="_blank">PCF</a> or <a href="http://run.pivotal.io/" target="_blank">PWS</a>.
<!--more-->

There's a known and recommended way to push GO applications. It is based on using <a href="http://docs.run.pivotal.io/buildpacks/go/index.html" target="_blank">GO buildpack</a>. Very briefly, the buildpack takes all of your source code and dependencies (managed by <a href="https://github.com/tools/godep" target="_blank">godep</a>), builds the binary and creates a droplet that is then distributed to CF nodes. The alternative approach I want to describe here is based on building the binary locally and push the binary instead of the source code. I'll leave the discussion about the advantages or disadvantages of such approach to a reader as in my opinion it is a matter of preference and specific project requirements.
<h2>Buildpacks</h2>
To understand what we're going to do, first we need to cover the idea of buildpacks. As per the <a href="http://docs.cloudfoundry.org/buildpacks/" target="_blank">documentation</a> <i>"buildpacks provide framework and runtime support for your applications. Buildpacks typically examine user-provided artifacts to determine what dependencies to download and how to configure applications to communicate with bound services."</i> 

And the description continues: <i>"When you push an application, Cloud Foundry automatically detects which buildpack is required and installs it on the Droplet Execution Agent (DEA) where the application needs to run."</i>.

In our case, we'll use a special buildpack, called <a href="https://github.com/igm/noop-buildpack" target="_blank">"noop buildpack"</a>. This buildpack (as its name suggests) does nothing. We don't need any special logic to introspect our application and find out what language it is written in or which runtime it requires, because we know it is going to be a Linux x64 binary. 

<h2>Application bootstrap</h2>
Now that we've effectively deactivated standard "boostrap" procedure that is added to an app if using any of the proper buildpacks, we need some other way to start our process. And we can, there's a special directory <code>".profile.d"</code> that CF recognises and executes all the scripts found there, like <code>"setenv.sh"</code> described <a href="http://docs.cloudfoundry.org/devguide/deploy-apps/deploy-app.html#profiled" target="_blank">here</a>. And this is exactly what we'll use to bootstrap the app. We'll create a new project with <code>".profile.d"</code> directory containing a script called <code>"start.sh"</code>.

<pre class="terminal">.
└── .profile.d
    └── start.sh
</pre>

I am sure a careful reader just realised that this is by no means GO specific. Any script can be executed there, for example a script to run a python http server for serving static pages (full example <a href="https://github.com/igm/static-files-demo" target="_blank">here</a>):

<script src="https://gist.github.com/igm/d0d6cc3f51619c29d9e1.js"></script>
The important part of the script is the <code>"cd app"</code>. During the staging process, all of your application artifacts that are pushed to Cloud Foundry are placed into the "app" directory as can be seen using <code>"cf files APP_NAME"</code> command:

<pre class="terminal">% cf files static-files-demo
Getting files for app static-files-demo in org ************* / space development as *************@gmail.com...
OK

.bash_logout                              220B
.bashrc                                   3.6K
.profile                                  675B
app/                                         -
logs/                                        -
staging_info.yml                          168B
tmp/                                         -
</pre>

<h2>GO application</h2>
Now I suppose all should be clear. We'll build local binary of our application. It does not really matter what method you use for managing dependencies, it can be <a href="https://github.com/tools/godep" target="blank">godep</a>, <a href="http://labix.org/gopkg.in" target=
blank">go.pkg</a> or <a href="http://getgb.io/" target="_blank">gb</a>. Just make sure to build amd64 linux binary:
<pre class="terminal">$ GOARCH=amd64 GOOS=linux go build APP
</pre>

Then create a <code>".profile.d/start.sh"</code> file with a content similar to this:
<script src="https://gist.github.com/igm/2d8e71928746b6ef6462.js"></script>
And push your application to CF using noop-buildpack:
<pre class="terminal">$ cf push APP_NAME -b https://github.com/igm/noop-buildpack
</pre>
<h2>Conclusion</h2>
This approach describes an alternative and not very often presented way to push application to Cloud Foundry. But in some situation this can be useful and really fast way to get your GO application running on Cloud Foundry or any of its flavours. The possible drawback is that it is assumed the PCF nodes run amd64 Linux OS (<a href="https://github.com/cloudfoundry/stacks" target="_blank">cflinuxfs2 derived from Ubuntu 14.04</a>), which is true at this time, but does not need to hold true in the future. 
