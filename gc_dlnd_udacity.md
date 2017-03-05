# Google cloud engine instance setup to train deep neural networks.

## Server Configuration

### After sign up to Google Cloud Platform, there are 3 steps to complete:

* Create a Linux-based VM instance with the required hardware specs. Tutorial [here](https://cloud.google.com/compute/docs/quickstart-linux)
* Install Software: Anaconda Python, Tensorflow and Julia.
* Set up Jupyter (IPython), so that you can do your MachineLearning/DataScience magic remotely via a browser.


### 1. Create a Linux VM Instance

GCE_CreateInstanceFollow the Quickstart guide to create a new VM instance, but note the following:

* Machine type: a micro instance isn’t going to cut it for compute intensive tasks. I created a 16 vCPU machine; select what works for you. Note: if you need a machine with more than 24 cores, you’ll need to increase your quota.
* Boot Disk: I’m more familiar with Ubuntu so, that’s what I picked (14.04 LTS). The setup instructions below assume you’re using Ubuntu.
* Firewall: Allow HTTPS traffic.
* Take note of the Zone and instance Name. You’ll need those them later in our final step. In this example, the zone is us-central1-f and the name is awesomeness

![google-console-image](https://haroldsoh.files.wordpress.com/2016/04/gce_createinstance.png?w=255&h=369)

### The SSH Browser-based Terminal

Google’s Compute Engine has a sweet browser-based SSH terminal you can use to connect to your machine. We’ll be using it for additional setup below.
![gcm-image2](https://haroldsoh.files.wordpress.com/2016/04/ssh_button_gce.png?w=1154)

### 2. Install Required Software

We’ll install two major software packages: Anaconda Python and Google Tensorflow. 
Bring up your SSH terminal and enter
```
wget http://repo.continuum.io/archive/Anaconda3-4.0.0-Linux-x86_64.sh
bash Anaconda3-4.0.0-Linux-x86_64.sh
```

and follow the on-screen instructions. The defaults usually work fine, but answer yes to the last question about  prepending the install location to PATH:
 ```
 Do you wish the installer to prepend the 
Anaconda3 install location to PATH 
in your /home/haroldsoh/.bashrc ? 
[yes|no][no] >>> yes
 ```

To make use of Anaconda right away, source your bashrc   

```
source ~/.bashrc
```

### Install Tensorflow and tqdm modules
```
conda install -c conda-forge tensorflow
conda install -c conda-forge tqdm
```
### 3. Set up Jupyter (IPython)

In our final step, we’ll need to set up the Jupyter server and connect to it. The following instructions come mainly from [here](https://cloud.google.com/dataproc/tutorials/jupyter-notebook), with some tweaks.

#### Open up a SSH session to your VM and generate the Jupyter configuration file
```
jupyter notebook --generate-config
```

#### We’re going to add a few lines to your Jupyter configuration file; the file is plain text so, you can do this via your favorite editor (e.g., vim, emacs), I used nano:
```
nano ~/.jupyter/jupyter_notebook_config.py
```

#### Add the following lines to config file
```
c = get_config()
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.port = 8123
```

Once that’s done we can start the server using **[screen](https://www.gnu.org/software/screen/manual/screen.html)**.

#### Install screen:
```
sudo apt-get install screen
```

and start a screen session with the name jupyter:
```
screen -S jupyter
```
The -S option names our session (else, screen will assign a numeric ID). I’ve chosen “jupyter” but the name can be anything you want.

Create a notebooks directory and start the jupyter notebook server:
```
cd ~/
mkdir notebooks
cd notebooks
jupyter notebook

```
Press CTRL-A, D to detach from the screen and take you back to the main command line. If you want to re-attach to this screen session in the future, type:

```
screen -r jupyter
```
You can now close your SSH session if you like and Jupyter will keep running.

## Client Configuration

Now that we have the server side up and running, we need to set up a SSH tunnel so that you can securely access your notebooks.

For this, you’ll need to install the [Google Cloud SDK](https://cloud.google.com/sdk/) on your local machine. Come back after it’s installed.

Now, authenticate yourself:
```
gcloud --init
```

and initiate a SSH tunnel from your machine to the server:
```
gcloud compute ssh  --zone=<host-zone> --ssh-flag="-D" --ssh-flag="1080" --ssh-flag="-N" --ssh-flag="-n" <host-name>
```

You’ll need to replace the **host-zone** and **host-name** with the appropriate zone and host-name of your VM (that you took note of in the first step). You’ll also find the relevant info on your [VM Instances page](https://console.cloud.google.com/compute/instances).

#### Finally, start up your favorite browser with the right configuration:
```
<browser executable path> --proxy-server="socks5://localhost:1080" --host-resolver-rules="MAP * 0.0.0.0 , EXCLUDE localhost" --user-data-dir=/tmp/
```

Replace **browser executable path** with your full browser path on your system. See [here](https://cloud.google.com/dataproc/tutorials/jupyter-notebook#configure_your_browser) for some common executable paths on different operating systems.

Finally, using the browser that you just launched, head to your server’s main notebook page:
```
http://<host-name>:8123
```
### Tip : Use instance name not IP (awesomeness in my case).
and if everything went according to plan, you should see something like this:
![image-3](https://haroldsoh.files.wordpress.com/2016/04/jupyter_tree.png?w=1024&h=684)

## That’s it! Congratulations!

### I just made some changes to make it compatible with dlnd projects.
### Sources
https://haroldsoh.com/2016/04/28/set-up-anaconda-ipython-tensorflow-julia-on-a-google-compute-engine-vm/
https://github.com/jg1141
https://github.com/xadahiya/gcm-tensorflow-udacity/blob/master/README.md
