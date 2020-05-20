
# An Introduction to DataPower with Docker

This lab will introduce you to the basics fo working with DataPower using Docker containers. You will learn how to launch and configure DataPower containers, as well we keep all the important assets in change control. You will also publish your final DataPower container image to a private repository, where it can be used by other team members or devops pipelines.



# Pre-requisites
## Knowledge
This lab is more about Docker than it is DataPower. No experience is required in either tool. You should be comfortable working in the command line.

## The computer you will be doing this lab on needs
* Linux based operating system with a GUI, a user with sudo/root access, and an Internet connection. A virtual machine will work fine and is encouraged.
* Docker installed. (https://docs.docker.com/engine/install/)
* Git installed. (https://git-scm.com/downloads)
* A GitHub account. (https://github.com/)
* A Docker Hub account. (https://hub.docker.com/)








# Git Setup
STAR
WHY?


In you terminal, verify that Git is installed with the following command:
````console
git --version
````
You will hopefully see something like the following to show what version is installed.
````console
git version 1.8.3.1
````

In your borwser, log into your github account and navigate to the repository that this lab is in: https://github.com/horvatca/IntroToDataPowerOnDockerLab.
Fork the repository. The Fork button can be found in the top right of the page when viewing the repository. 

#insert FORK pick

This will give you yor own copy to work with in your own GitHub account and make setting up the lab environment easier. Navigate to the reposity in YOUR account.
Next, download YOUR repository to YOUR local home directory. The below commands move you to your home directory and downlaod, or "clone", the repository. In your git clone command, use the link to your repository. The link can be found in the green "Clone or download" button when viewing YOUR repository in YOUR account in the browser.


#insert CLONE button

````console
cd ~
git clone https://github.com/[YOUR ACCOUNT NAME HERE]/IntroToDataPowerOnDockerLab.git
````

Now you have a copy of this lab on your local computer.





# Docker Setup
STAR

### Verify Docker installation.
In you terminal, verify that Docker is installed with the following command:
````console
docker -v
````
You will hopefully see something like the following to show what version is installed.
````console
Docker version 19.03.8, build afacb8b
````

### Run your first Docker container.

In you terminal, run the following command:
````console
docker run hello-world
````
You will see something like the following to show what version is installed.
````console
Docker version 19.03.8, build afacb8b
````
Great! Docker is working!


# Starting a DataPower container
STAR
You now want to run your first DataPower container. Docker makes this super easy. You can do it with a single command.

First, make a directory to work in. Do this inside of your local repository. Open up it's permissions so that our DataPower can write to it later, then enter it.
````console
mkdir ~/IntroToDataPowerOnDockerLab/mydp
sudo chmod 777 -R ~/IntroToDataPowerOnDockerLab/mydp
cd ~/IntroToDataPowerOnDockerLab/mydp
````

Second, start a DataPower container. In you terminal, run the following command:
````console
docker run -it \
  -v $PWD/config:/drouter/config \
  -v $PWD/local:/drouter/local \
  -e DATAPOWER_ACCEPT_LICENSE=true \
  -e DATAPOWER_INTERACTIVE=true \
  -e DATAPOWER_WORKER_THREADS=4 \
  -p 9090:9090 \
  ibmcom/datapower:2018.4.1
````
You will see something like the following to show what version is installed.
````console
Unable to find image 'ibmcom/datapower:2018.4.1' locally
2018.4.1: Pulling from ibmcom/datapower
acb220039030: Pull complete 
... ... ... a bunch of logs that arent important right now ... ... ...
20200511T192821.747Z [0x8100003b][mgmt][notice] domain(default): Domain configured successfully.
20200511T192822.372Z [0x00350014][mgmt][notice] quota-enforcement-server(QuotaEnforcementServer): tid(799): Operational state up
login: 
````

>What did that command do?
>You just provisioned a DataPower with a single command! Wasn't that cool? Way easier and faster than other methods? I thnk so!
>You asked Docker to run a DataPower container and it did! Let's break down the command line by line:
>docker run -it    -This tells docker to run the container with a bash shell (terminal) attached to it
>-v $PWD/config:/drouter/config     -This binds a volume inside the container to a volume on your local OS. More on this later in the lab.
>-v $PWD/local:/drouter/local     -This binds a volume inside the container to a volume on your local OS. More on this later in the lab.
>-e DATAPOWER_ACCEPT_LICENSE=true     -This sets an environent varialbe for the container to a specific value. In this case, it accepts the license agreement.
>-e DATAPOWER_INTERACTIVE=true      -This sets an environent varialbe for the container to a specific value. In this case, it allows acess to the container via the command line after it is started.
>-e DATAPOWER_WORKER_THREADS=4     -This sets an environent varialbe for the container to a specific value. In this case, it sets the number of worker threads to 4.
>-p 9090:9090      -This maps port 9090 of the container to port 9090 of your host OS.
>ibmcom/datapower:2018.4.1     -This specifies that we want to run the container named "datapower" from the "/ibmcom" repository. Specifically the "2018.4.1" version.
>
>Another thing to note is that docker downloaded the container image. If looks to see if you have the image locally first, then downlaods it if it is not present. Once it has the image, it creates a container based on the image. That image can be used over and over again to create conainers. Think of it like  a teamplate.


### Using your new DataPower container
DataPower is running and now you want to configure it to do something! This lab is not meant to teach DataPower configurations, so we will keep it simple. We will log in, make some configurations, and then export the configuration.

In you terminal, log into the runnign DataPower. The login promp should be present already since you started the container. The super secret default user is "admin" and the super secret default passord is.... "admin".
````console
login: admin
Password: *****
````
The output should be similar to the following:
````console
Welcome to IBM DataPower Gateway console configuration. 
Copyright IBM Corporation 1999, 2020 

Version: IDG.2018.4.1.10 build 318002 on Feb 21, 2020 11:09:49 AM
Delivery type: LTS
Serial number: 0000001

idg# 
````


The first thing you probalby want to do if you are doing DP dev work is enable the Web GUI.
In you terminal, run the following command:
````console
configure; web-mgmt 0 9090 9090;
````
The output should be similar to the following:
````console
Global mode
Web management: successfully started
````
>Remember in the docker run command when we mapped the container's port 9090 to the host's port 9090? Well we just started the web GUI on the container's port 9090, so now we should be able to access it from our host's port 9090.

Open up a web browser and browser to https://localhost:9090.
You should see the DataPower login screen.
# Insert DP login screen image.



# Make and save some configurations
Now that we have access to the GUI, we can make some configurations. Then we will export them so that we can build them into a container image and rapidly launch LOTS of DataPower containers with those configurations.

If you arent already there, open up a web browser and browser to https://localhost:9090. Then, login with the un/p of admin/admin again. Leave the other settings alone. 

#insert empty GUI image

Now that you are logged in, you can see that there is nothign configured in this DP. That is because it was made from the the bare DP image. Not it's time to make some configurations so that we


Let's make a few simple configurations.

####Create a new domain
In the left most menu, click on the Admin gear, then click the Configuration drop down arrow, then select Application Domain. Your screen will look like the below image.

#insert new domain 1 image

Click the New... button and give your domain a name and a comment like in the below image. Then click Apply.

#insert new domain 2 image

You will be taken to the Applicain Domains list and yours will appear like in the below image.

#insert new domain 3 image.

#### Create a new Multi Protocol Gateway
In the left most menu, click on the Services cloud, then click the Multi-Protocol Gateway option.d Your screen will look like the below image.

#insert new MPGW 1

Click the New button.

........


#### Create a new Web Service Proxy
In the left most menu, click on the Services cloud, then click the Multi-Protocol Gateway option.d Your screen will look like the below image.

#insert make_web_service_proxy.png

Click the New button.

.....



#### Save the configurations

Across the top of the DP interface in the browser has been a message bar saying "The running configuration of the device contains unsaved changes." Click the Save Changes link in that message bar.

#insert messagebar image

#### Explore the configuration files

The changes you just made to the running DataPower (made a Web Service Proxy and a new Application Domain) were persisted in configuration files. Remember the  "-v $PWD/config:/drouter/config" and "-v $PWD/local:/drouter/local" parts of the command that we used to start the container? Those mapped where the ocnfiguraiton files are saved to our local host file system. Let's explore them with the following commands.


````console
tree ~/IntroToDataPowerOnDockerLab/mydp
````
What you see are two directories inside the mydp directory where you started the container from. config and local were created by the container to persist the settings.

> Note: If you don't have tree installed and dont want to install it on your machine, don;t worry, you can just look at the below screen shot. Not using the tree command will not affect the rest of this lab. It is only a nice visual. You could also explore the files with the ls command.

# insert tree_mydp.png image


Now lets move into the config directory and take a look at the .cfg files in it. We will search the auto-startup.cfg file for the test "My" so that we will be shown the exact lines that reference the "MyNewWebServiceProxy" and "MyNewAppDomain" we created.

````console

cd  ~/IntroToDataPowerOnDockerLab/mydp/config
sudo grep .*My.* auto-startup.cfg 
````
You'll see something like the below. LOOK! OUR STUFF!
# insert auto_startup_grep.png image

> Note: You can also check out the entire text if you want, but it's long. The command is "sudo cat auto-startup.cfg"


For the changes we made, this auto-startup.cfg file is the only place they will show up.


### Put your configuratioins in change control using Git.

We want to be able ot use these configurations over and over, see what changes from version to version, and share them with team members. The best place do that is in a Git repository. We will "push" these files to our git repo now.
>Note: Most GitHub repositories like the ones we are using are pulic, and this anybody can see. In a real scenario, you would have your own private repository that allowed you to control access. 


NOTE: May have to stop the container. During testing, try with it running. I wrote the below with it stopped.
[ibmuser@localhost IntroToDataPowerOnDockerLab]$ docker ps
CONTAINER ID        IMAGE                       COMMAND             CREATED             STATUS              PORTS                    NAMES
a868e8dd216b        ibmcom/datapower:2018.4.1   "/start.sh"         9 days ago          Up 9 days           0.0.0.0:9090->9090/tcp   mystifying_wright
[ibmuser@localhost IntroToDataPowerOnDockerLab]$ docker stop a868e8dd216b
a868e8dd216b
END NOTE

Use the below commands to push your ocnfiguraiton file to your ghithub repository.


````console
cd ~/IntroToDataPowerOnDockerLab
git add .
git commit -m 'Adding the dp configurations.'
git push
````

Now browse to your repository. You should see that in addition to the README.md, your configs are also there! It should look like the below screen shot:

#add image github_with_pushed_configs.png 

> Note: If you explore the files that got pushed to github, you will notice that the /IntroToDataPowerOnDockerLab/mydp/local directory was no pushed. This is beause the directory is empty and by default, Git will ignore it. This is OK. It will not interfere with our lab.

WONDERFUL! You now have the configurations you made to DataPower stored in version control! In the next parts of the lab, we will use them to create and deploy more DataPowers with the same configurations.



## Launch another DataPower with the configurations



XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX



430 tom
330 friday





### TEMPLATE
STAR

In you terminal, run the following command:
````console

````
The output should be similar to the following:
````console

````
