# Google cloud instance setup to train deep neural networks.

### After sign up to Google Cloud Platform, there are 3 steps:

* Create a Linux VM instance with required hardware specs. Tutorial [here](https://cloud.google.com/compute/docs/quickstart-linux)
* Install Software: Anaconda Python, Tensorflow and Julia.
* Set up Jupyter (IPython), so that you can access remotely via a browser.

### 1. Create a Linux VM Instance

GCE_CreateInstanceFollow the Quickstart guide to create a new VM instance, but note the following:

* Machine type: a micro instance will not suffice for compute intensive tasks. Can create 16 vCPU machine, if you need a machine with more than 24 cores, you’ll need to increase your quota.
* Boot Disk: I chose Ubuntu.
* Firewall: Allow HTTPS traffic.
* Take note of the Zone and instance Name. You’ll need those them later in our final step.

![google-console-image](https://haroldsoh.files.wordpress.com/2016/04/gce_createinstance.png?w=255&h=369)

### The SSH Browser-based Terminal

Google’s Compute Engine has a browser-based SSH terminal you can use to connect to your machine.
![gcm-image2](https://haroldsoh.files.wordpress.com/2016/04/ssh_button_gce.png?w=1154)

### 2. Install Required Software

In SSH terminal enter
```
wget http://repo.continuum.io/archive/Anaconda3-4.0.0-Linux-x86_64.sh
bash Anaconda3-4.0.0-Linux-x86_64.sh
```
answer yes to the last question about prepending the install location to PATH:
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

Set up Jupyter server. The following instructions come mainly from [here](https://cloud.google.com/dataproc/tutorials/jupyter-notebook), with some tweaks.

#### Open up a SSH session to your VM and generate the Jupyter configuration file
```
jupyter notebook --generate-config
```

#### Add below lines to Jupyter configuration file use any eitor you like (e.g., vim, emacs, nano):
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

For this, you’ll need to install the [Google Cloud SDK](https://cloud.google.com/sdk/) on your local machine.
After it is installed, authenticate yourself:
```
gcloud --init
```
Then initiate a SSH tunnel from your machine to the server:
```
gcloud compute ssh  --zone=<host-zone> --ssh-flag="-D" --ssh-flag="1080" --ssh-flag="-N" --ssh-flag="-n" <host-name>
```

Replace the **host-zone** and **host-name** with the appropriate zone and host-name of your VM. You’ll also find the relevant info on your [VM Instances page](https://console.cloud.google.com/compute/instances).

#### Finally, start up your browser with the right configuration:
```
<browser executable path> --proxy-server="socks5://localhost:1080" --host-resolver-rules="MAP * 0.0.0.0 , EXCLUDE localhost" --user-data-dir=/tmp/
```

Replace **browser executable path** with your full browser path on your system. See [here](https://cloud.google.com/dataproc/tutorials/jupyter-notebook#configure_your_browser) for some common executable paths on different operating systems.

Finally, using the browser that you just launched, head to your server’s main notebook page:
```
http://<host-name>:8123
```
### Tip : Use instance name not IP.

### Sources
https://haroldsoh.com/2016/04/28/set-up-anaconda-ipython-tensorflow-julia-on-a-google-compute-engine-vm/
https://github.com/jg1141
https://github.com/xadahiya/gcm-tensorflow-udacity/blob/master/README.md
