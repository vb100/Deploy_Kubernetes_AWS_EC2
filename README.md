<h1>Set up Kubernetes on AWS EC2 Instance</h1>
<p>Prepared by Vytautas Bielinskas, 2021.<br>
Video of steps explained: <a href="https://www.youtube.com/c/VytautasBielinskas">Set up Kubernetes on AWS EC2 Instance</a><br>LinkedIn profile: <a href="https://www.linkedin.com/in/bielinskas/">here</a></p>

---

<h2>Prerequisites</h2>
<p>To sucessfully complete this tutorial you should: 
<ul>
  <li>have an active AWS Account.</li>
  <li>basic understanding of Unix terminal commands.</li>
  <li>basic understanding of Docker.</li>
</ul>
If you say yes to all these requirements, we can start.
</p>

<h2>How to read this tutorial?</h2>
<p>This tutorial provides clear steps and commands which must be executed sequentially as it is listed. I strongly recommend not to skip any step, except <i>Workaround</i> step if all forward commands will no raise any errors.</p>

<h2>Hands on set up your AWS EC2 Instances</h2>
<p>In this part we will closely work on initialzied AWS EC2 instances. Keep in mind that in order to complete this part sucessfully, you should be opened all 3 terminal windows during all steps. Here 1 terminal window is dedicated to <b>Master slave</b>, while other two to <b>Worker nodes</b>. So in total we are having 3 nodes (1 + 2).<br>Noticing that this manual was prepared in Mac, so slightly differences may be using different OS, such as Windows.</p>
  
---
<h3>Prepare your SSH Private key for safe connections</h3>
<p>First of all, put you SSH key to a special folder in your machine. Keep SSH key filnename be easy to type and recognize such as <i>kubernetes-dev.pem</i> (we will use this filename furthermore in this manual).</p>
<p>Before all forwarding steps, you must to allow your <i>.pem</i> file be executable in your system. To do this, type and enter following command:</p>

```
chmod 400 kubernetes-dev.pem 
```

As a result you should see granted access to your <i>.pem</i> file - type <code>ls -la</code> command to see the effect:

```commandline
-r--------@    1 usename  staff   1700 Nov 16 07:14 kubernetes-dev.pem
```
This setting ensure safe connection to instances to execute next steps. 
</p>

<h3>Install dependencies and configure Kubernetes</h3>
<p>
In this part we will install Docker and Kubernetes packages into all 3 nodes, then enable them to work. In addition to that we will prepare Kubernetes tools such as <code>kubectl</code>, and <code>kubeadm</code> once they are mandatory for set up.
</p>
<p><b><u>Important:</b> Following steps must be executed in all your nodes, included Master and Worker nodes in paraller.</u></p>

<p><b>Step 1. Install Docker</b><br>
In order to install Docker package to your node, type and enter following commands:

```commandline
# Update the package list
sudo apt-get update
# Install Docker
sudo apt-get install docker.io
# Check the installation and Docker version
docker --version
```
</p>

<p><b>Step 2. Start and enable Docker</b><br>
Once you installed Docker in a previous step, you should to start and enable for a work it by type and enter following commands.
  
```commandline
# Set Docker to launch at boot
sudo systemctl enable docker
# Verify that Docker is running now
sudo systemctl status docker
# If not running, try this:
sudo systemctl start docker
```
</p>

<p><b>Step 3. Install Kubernetes</b><br>
To have our instances set up correctly, by side of Docker, we must to install a Kubernetes service. Kubernetes used to orchestrate Docker images, scale up and down based on real-time demand, and it also has number of benefits. Since you are downloading Kubernetes from a non-standard repository, it is essential to ensure that the software is authentic. So we need to add a signing key (with extenstion <code>gpg</code>).<br>
To install Kubernetes type and enter the following command.
  
```commandline
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
```
  
Once you sucessfully added a signing key, you have to ass software repository with the following command:
  
```commandline
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```
</p>

<p><b>Step 4. Prepare Kubernetes installations tools</b><br>
  Here we will install <code>kubeadm</code>, <code>kubelet</code>, and <code>kubectl</code> tools. These tools are required for cluster initializing and managing. To install all these tools, type and enter following commands:
  
```commandline
sudo apt-get install kubeadm kubelet kubectl
sudo apt-mark hold kubeadm kubelet kubectl  
```
  
When you are completed the previous command, be sure that the installations was sucessfull. To do it, verify the installation and check the version of installed <code>kubeadm</code> with the following command:
  
```commandline
kubeadm version
```
If you done all previous steps correctly, no any erros should raises. You should see some version data and metadata related to your Kubenetes version.
</p>

<h3>Kubernetes Deployment</h3>
<p>With this part we will begin the deployment process for freshly installed Kubernetes on our EC2 instances for all nodes. Keep executre following commands for all 3 nodes.</p>

<p><b>Step 1. Disable the swap memory from each node</b><br>
Before initializing the internal Kubernetes network on our instances, you must the disable the swap memore from each node with the following command:

```commandline
sudo swapoff -a
```
No any message should apper after executing the previous command. If no an error on your side as well, keep going.
</p>

<p><b>Step 2. Set a host name for each node</b><br>
Here we need to set meaningful host names for each of node. The purpose of this step is to easily recognize which node is Master, ans which ones are Worker nodes in our deployment processes. You should set host names to each instance separately:
  
<ul>
  <li>For <b>Master</b> node: <code>sudo hostnamectl set-hostname master-node</code></li>
  <li>For <b>Worker</b> node 1: <code>sudo hostnamectl set-hostname worker-1</code></li>
  <li>For <b>Worker</b> node 2: <code>sudo hostnamectl set-hostname worker-2</code></li>
</ul>
You can set host names as you want, once there are no any strictly rules on that. Just keep in mind that the names should be meaningful, it will make you life easier for later.
</p>

<p><b>Step 3. Workaround: to avoid drivers mistmatches</b><br>
  
  > <b>Important</b>: this step is optional as some instances raises error due to mismatches between installed drivers in the next steps. I recommend to apply this workaround to avoid such issues.
  
  The main idea of this workaround is to create <i>daemon.json</i> file with specified content inside in <i>/etc/docker/</i> dircetory to avoid mismatches among drivers and make Kubernetes run healthly. You can do it with the following commands on all instances (nodes) terminals:
  
```commandline
cd ../../etc/docker
sudo touch daemon.json
```
  
  Once you have created <i>daemon.json</i>, you must to fill this file with specified content inside as below. First of all, open this JSON file for editing with <code>vi</code> command:
  
```commandline
sudo vi daemon.json 
```
  
Once you are entered to editing space, type (or copy and paste) this conent:
  
```commandline
{
"exec-opts": ["native.cgroupdriver=systemd"]
} 
```
  
Be sure that you pasted this content correctly, and exit with save the file with the hotkey <code>:wq</code>.
  
</p>
