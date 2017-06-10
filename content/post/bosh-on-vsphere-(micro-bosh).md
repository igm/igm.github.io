+++
date = "2012-06-28"
draft = true
title = "BOSH on vSphere (micro Bosh)"

+++
Recently I spent some time playing with BOSH which is a tool for <a href="https://github.com/cloudfoundry/bosh" target="_blank">release engineering, deployment and lifecycle management of large scale distributed services</a>. It is used to manage VMs in AWS and vSphere. The vSphere option seemed to be interesting to me and the idea of having private <a href="http://cloudfoundry.org/" target="_blank">Cloud Foundry</a> PaaS running locally on private infrastructure was exciting. The BOSH is under heavy development and it's sometimes difficult to make things work easily even if following various tutorials and <a href="http://drnicwilliams.com/2012/04/16/creating-a-bosh-from-scratch-on-aws/" target="_blank">blogs</a> and I'm very thankful to the authors.

<!--more-->
## vSphere Setup
As an infrastructure I created a cluster of 4 hosts ESXi version 5u1 (esxi1, esxi2, esxi3, esxi4) registered in igmlab.net domain (i.e. esxi1.igmlab.net). All of the hosts were powered by four core CPU wih 16GB of RAM and iSCSI shared storage (IET software based) of 512GB. The internal network had following characteristics: IP range: 192.168.2.0/24, GW/DNS: 192.168.2.107. For <a href="http://www.vmware.com/products/vcenter-server/overview.html" target="_blank">VMware vCenter Server</a> SUSE based VM appliance was used (verion 5.0.0).

Proper network configuration is crucial for correct deployment. Bosh VMs run as guests in ESXi hosts and they access those hosts directly so both the hosts and guests have to  share the same network. This is also the case for vCenter Server.

{{< figure src="/images/bosh-on-vsphere-(micro-bosh)/image001.png" width="400px" >}}
{{< figure src="/images/bosh-on-vsphere-(micro-bosh)/image003.png" width="400px" >}} </br>
{{< figure src="/images/bosh-on-vsphere-(micro-bosh)/image005.png" width="400px" >}}
{{< figure src="/images/bosh-on-vsphere-(micro-bosh)/image007.png" width="450px" >}}

<!-- <a href="http://1.bp.blogspot.com/-2karvJbVftY/T-x7jufBU_I/AAAAAAAAHXA/1hlRtfY-WzU/s1600/image001.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="191" src="http://1.bp.blogspot.com/-2karvJbVftY/T-x7jufBU_I/AAAAAAAAHXA/1hlRtfY-WzU/s400/image001.png" width="400" /></a>   <a href="http://4.bp.blogspot.com/-uobwxIfuGLs/T-x7nnthoKI/AAAAAAAAHXI/3Cc42WtR5ig/s1600/image003.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="191" src="http://4.bp.blogspot.com/-uobwxIfuGLs/T-x7nnthoKI/AAAAAAAAHXI/3Cc42WtR5ig/s400/image003.png" width="400" /></a>  <a href="http://4.bp.blogspot.com/-h0f5TNAUTf4/T-x7wp00aDI/AAAAAAAAHXY/leiMecqlMJA/s1600/image005.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="191" src="http://4.bp.blogspot.com/-h0f5TNAUTf4/T-x7wp00aDI/AAAAAAAAHXY/leiMecqlMJA/s400/image005.png" width="400" /></a>   <a href="http://1.bp.blogspot.com/-ZwTIKafeAKo/T-x7zTts_2I/AAAAAAAAHXg/33Lm4S3Sj34/s1600/image007.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="191" src="http://1.bp.blogspot.com/-ZwTIKafeAKo/T-x7zTts_2I/AAAAAAAAHXg/33Lm4S3Sj34/s400/image007.png" width="400" /></a>   -->

## Micro Bosh
To deploy Bosh you need Bosh. It's kind of like to get a compiler you need to compile one first. Luckily there's a special one called "micro Bosh" and it can be installed by Bosh CLI addon called Bosh Deployer. So first things first, let's start with <a href="http://rubygems.org/gems/bosh_cli" target="_blank">Bosh CLI</a>. The current version at the time of writing is 0.9.14. Ruby people should not have any problems installing it via <code>gem install bosh_cli</code>. More detailed instructions are <a href="https://github.com/cloudfoundry/oss-docs/blob/master/bosh/documentation/documentation.md#installing-bosh-command-line-interface" target="_blank">here</a>. The next step is to install <a href="https://github.com/cloudfoundry/oss-docs/blob/master/bosh/documentation/documentation.md#bosh-installation" target="_blank">Bosh Deployer</a>. For trouble-less installation <a href="http://releases.ubuntu.com/lucid/" target="_blank">Ubuntu 10.04.4 LTS Server</a> is suggested. The steps: 

	$ gem install bosh_cli
	$ git clone https://github.com/cloudfoundry/bosh.git
	$ cd bosh/deployer
	$ bundle install
	$ rake install
	$ rbenv rehash
	$ bosh
	usage: bosh [--verbose] [--config|-c &lt;FILE&gt;] [--cache-dir &lt;DIR]
	            [--force] [--no-color] [--skip-director-checks] [--quiet]
	            [--non-interactive]
	            command [&lt;args&gt;]
	...
	Micro
	  micro deployment [&lt;name&gt;] Choose micro deployment to work with 
	  micro status              Display micro BOSH deployment status 
	  micro deployments         Show the list of deployments 
	  micro deploy &lt;stemcell&gt;   Deploy a micro BOSH instance to the currently 
	                            selected deployment 
	                            --update   update existing instance 
	  micro delete              Delete micro BOSH instance (including 
	                            persistent disk) 
	  micro agent &lt;args&gt;        Send agent messages 
	  micro apply &lt;spec&gt;        Apply spec &nbsp;

Next we need micro bosh <a href="https://github.com/cloudfoundry/oss-docs/blob/master/bosh/documentation/documentation.md#deployments" target="_blank">deployment</a> manifest. In the manifest the deployment configuration is stored. The defaults for vSphere can be found in the <a href="https://github.com/cloudfoundry/bosh/blob/master/deployer/config/vsphere_defaults.yml" target="_blank">repository</a> and the manifest for micro Bosh deployment looks like this:

<script src="https://gist.github.com/3002576.js?file=micro_bosh.yml">
</script> 
We need to create deployments directory to store our deployment manifest files. 
<pre>$ mkdir -p ~/deployments/micro
$ cd ~/deployments/micro
$ vim micro_bosh.yml
$ ... copy and update the content of bosh_micro.yml file
</pre>
Micro Bosh Stemcell is a VM template that will be used for our deployment. Deployment manifests define the details that are configured in the VM so that all the network configuration is set properly. It also contains details about the vSphere setup, vCenter connection parameters, resource parameters for VM etc. To get a full list of available Bosh stemcells run:

<pre>$ bosh public stemcells
+---------------------------------+-------------------------------------------------------+
| Name                            | Url                                                   |
+---------------------------------+-------------------------------------------------------+
| bosh-stemcell-0.3.0.tgz         | https://blob.cfblob.com/rest/objects/4e4e78bca41e1... |
| bosh-stemcell-0.4.4.tgz         | https://blob.cfblob.com/rest/objects/4e4e78bca51e1... |
| bosh-stemcell-0.4.7.tgz         | https://blob.cfblob.com/rest/objects/4e4e78bca21e1... |
| bosh-stemcell-0.5.2.tgz         | https://blob.cfblob.com/rest/objects/4e4e78bca31e1... |
| bosh-stemcell-aws-0.5.1.tgz     | https://blob.cfblob.com/rest/objects/4e4e78bca21e1... |
| bosh-stemcell-vsphere-0.6.0.tgz | https://blob.cfblob.com/rest/objects/4e4e78bca41e1... |
| bosh-stemcell-vsphere-0.6.1.tgz | https://blob.cfblob.com/rest/objects/4e4e78bca31e1... |
| micro-bosh-stemcell-0.1.0.tgz   | https://blob.cfblob.com/rest/objects/4e4e78bca51e1... |
+---------------------------------+-------------------------------------------------------+
To download use 'bosh download public stemcell <stemcell_name>'.For full url use --full.
</stemcell_name></pre>
At this moment we are interested in micro stemcell (micro-bosh-stemcell-0.1.0.tgz) and we need to download it: 
<pre>$ mkdir -p ~/stemcells
$ cd ~/stemcells
$ bosh download public stemcell micro-bosh-stemcell-0.1.0.tgz 
</pre>
As soon as we have required stemcell and manifest we can start the deployment procedure: 
<pre>$ cd ~/deployments
$ bosh micro deployment micro
$ bosh micro deploy ~/stemcells/micro-bosh-stemcell-0.1.0.tgz
Deploying new micro BOSH instance `micro/micro_bosh.yml' to `micro' (type 'yes' to continue): yes

Verifying stemcell...
File exists and readable                                     OK
Using cached manifest...
Stemcell properties                                          OK

Stemcell info
-------------
Name:    bosh-stemcell
Version: 0.5.1


Deploy Micro BOSH
  unpacking stemcell (00:00:26)                                                 
  uploading stemcell (00:06:21)                                                 
  creating VM from sc-d8ed6782-ff50-4e1d-ac47-519b225ba85a (00:09:02)           
  waiting for the agent (00:01:34)                                              
  create disk (00:00:00)                                                        
  mount disk (00:00:42)                                                         
  stopping agent services (00:00:00)                                            
  applying micro BOSH spec (00:01:12)                                           
  starting agent services (00:00:00)                                            
  waiting for the director (00:00:49)                                           
Done             11/11 00:20:41                                                 
Deployed `micro/micro_bosh.yml' to `micro', took 00:20:41 to complete
</pre>
Entire process takes several minutes (~20 minutes in my case).
<a href="http://3.bp.blogspot.com/-VAxjYJDcyzk/T-x70ekugHI/AAAAAAAAHXo/2hYP8O62bCo/s1600/image009.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="198" src="http://3.bp.blogspot.com/-VAxjYJDcyzk/T-x70ekugHI/AAAAAAAAHXo/2hYP8O62bCo/s400/image009.png" width="400" /></a>  <a href="http://4.bp.blogspot.com/-dhcpu4Oof9E/T-x71Loaz8I/AAAAAAAAHXw/sobXCjj9nXc/s1600/image011.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="200" src="http://4.bp.blogspot.com/-dhcpu4Oof9E/T-x71Loaz8I/AAAAAAAAHXw/sobXCjj9nXc/s400/image011.png" width="400" /></a>  
Now we can target the micro bosh and login with default admin/admin: 
<pre>$ bosh target 192.168.2.120:25555
$ bosh status
Updating director data... done

Target         micro (http://192.168.2.120:25555) Ver: 0.4 (00000000)
UUID           a5d6347d-3b66-465f-ae06-2444db70dce3
User           admin
Deployment     not set
</pre>
You should also be able to ssh to the micro Bosh: 
<pre>$ ssh root@192.168.2.120
root@192.168.2.120's password: vmware
</pre>
The password 'vmware' is what we provided in our manifest in env:bosh:password. You can also generate hash for custom password: 
<pre>$ mkpasswd -m sha-512
</pre>
and use it instead before deploying micro Bosh. And if you think this is all you wanted to do you can delete the deployment: 
<pre>$ bosh micro delete
</pre>

