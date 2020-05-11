
# An Introduction to DataPower with Docker

This lab will introduce you to the basics fo working with DataPower using Docker containers. You will learn how to launch and configure DataPower containers, as well we keep all the important assets in change control. You will also publish your final DataPower container image to a private repository, where it can be used by other team members or devops pipelines.



# Pre-requisites
## Knowledge
This lab is more about Docker than it is DataPower. No experience is required in either tool. You should be comfortable working in the command line.

## The computer you will be doing this lab on needs
* Linux based operating system with a GUI, root access and an Internet connection. A virtual machine will work fine and is encouraged.
* Docker installed. (https://docs.docker.com/engine/install/)
* Git installed. (https://git-scm.com/downloads)
* A GitHub account. (https://github.com/)
* A Docker Hub account. (https://hub.docker.com/)








# Git
workgin with git:
Create your git repo via web.

Now, set up the repo on your desktop so that you have a place to work and push assets to your repository. 

````console
#make the local directory for your repository (where you will put files you want to kepp in change control)
mkdir ~/IntroToDataPowerOnDockerLab
#move into that directoy
cd ~/IntroToDataPowerOnDockerLab
#Create a README.md for your repositoy. This is common best practice and also gives us something for our first push.
echo "# IntroToDataPowerOnDockerLab" >> README.md
#Initialize the current directory as a git repo.
git init
#Add the README.md file to the change control.
git add README.md
#Create your first commit. This is the set of changes you are goin to push to the repository.
git commit -m "First things first: add a README.md"
#Add the repository you created as 
git remote add origin https://github.com/horvatca/IntroToDataPowerOnDockerLab.git
git push -u origin master
````


Log into your github account and navigate to the repository that this lab is in: https://github.com/horvatca/IntroToDataPowerOnDockerLab.
For the repository. This will give you yor own copy to work with and make setting up the lab environmetn easier.
Next, download the repository. 
````console
git clone https://github.com/[YOUR ACCOUNT NAME HERE]/IntroToDataPowerOnDockerLab.git
````




I'm a dev, needing to do some DP development.

Pull the base DP image:
When you asdfsadfsadfasdfasdfasdf
docker pull ibmcom/datapower:latest
docker pull ibmcom/datapower:2018.4.1

See it in your local machine:
docker images -a
delete images:
docker rmi -f

Remove container:
docker container rm XXXXXID

Create and go to a directory you want to use for this DP launch and its stuff.
mkdir ~/DPonDocker1
cd ~/DPonDocker1

Start it:
Break downthis command:
DATAPOWER_INTERACTIVE=true prompts for login to the DataPower CLI on stdin and must be used with -it. This intermixes log output, disable DATAPOWER_LOG_STDOUT if not desired.
We recommend that you always set --worker-threads or DATAPOWER_WORKER_THREADS=x to a value less than the number of CPUs on your system. Otherwise, the DataPower Gateway will default to one worker thread for each CPU.


docker run -it \
  -v $PWD/config:/drouter/config \
  -v $PWD/local:/drouter/local \
  -e DATAPOWER_ACCEPT_LICENSE=true \
  -e DATAPOWER_INTERACTIVE=true \
  -e DATAPOWER_WORKER_THREADS=4 \
  -p 9090:9090 \
  ibmcom/datapower:2018.4.1

show logs.

Look! I'm at the DO login. This is a fresh new DP.
That was fast. It saved me a lot of time, but I want to go further and rapidly deploy MY datapowers
with MY configs in them. That is the real meat fo what we are doing today.

Before we login, lets look at some things.

open a new terminal
show the container running:

docker ps -a
tell them what they see.
port map, up time, name.


look at the file system and the DP folders.
There is now a config and a local folder.
/drouter/config is the location where DataPower will persist the configuration using an easy to read and editable format.
/drouter/local is used to store source files such as JavaScript (GatewayScript), XSLT, key, certificates and so on.
Changes to the contents fo these can be picked up by the DP and applied and visa versa, this is where configs you export will be stored.


back to DP.
login with admin/admin

first thing yo probalby want to do if you are doing DP dev work, enable the Web GUI:
configure; web-mgmt 0 9090 9090;

lets browse to that web GUI to verify
https://localhost:9090

You can see that this works, and because it was made from the the bare DP image, there is nothign here. It's brand new. The only real config we made was to enable this web interface.
Make a configuration....
Create a new domain - Admin gear > Config > App Domain > new button. It shows up in the top right drop down with default.
Create a MPGW - new service, mpgw,   you will ahe to create a new Front Side PRotocol.
make a https handler. call give it a name and set an ssl proxy, then save. click add under FSP settings.
Note that none of this will work, i jsut wan to show the concept of building an imutable container with settings.
Click the APPLY / SAVE button.
go to the config file and cat + grep the names of the things added to show they are in teh setgtings file.
Note that you should be putting these in change control - like git.

So I now have my DataPower configuration in a file, how do I transform that into a
DP container with those settings in it so that I can rapidly deploy and scale?
We are going to build an IMAGE, with those configs in it. We will then use that image
to create containers, that are the runnign DPs with the configs.
We define that image with the Docker BUILD file.


FROM ibmcom/datapower:2018.4.1

ENV  DATAPOWER_ACCEPT_LICENSE=true \
     DATAPOWER_WORKER_THREADS=2 \
     DATAPOWER_LOG_COLOR=false

EXPOSE 9090

COPY auto-startup.cfg /drouter/config/auto-startup.cfg

now we put the buidl file in the same folder as the auto-startup.cfg and execute the build.
More complicated DP configs may have more files to add to the foler and copy in, but they
are all done in the same way.

docker build -t mynewdp:1.0 .

this is creating the image we defined.
now let's run and test that image.

docker run -p 9090:909XXXXX mynewdp:1.0 --name firstDP

dosker ps -a to see the runnign containers.
docker stop <old container>

now lets go to the new one.

https://localhost:9090

notice I didnt have to run it interactive an enable the web interface?
my domain and MPGW are there. cool. lets put up another one!

docker run -p 9090:909XXXXX mynewdp:1.0 --name secondDP

docker ps

I can rapidly spin a lot of these up, put a load balancer in front of them, and im off
to the races. if somehtign happens and I lose my servers or move to a new hosting provider,
orhow bout, want to set up another enviornetn to test in, i can deploy more!
I can get more compicated and make it so that I can pass variables in as well. That makes
it vary powerful for deployign slighlty different configs and for automating the deployemtns with tools.
In fact, that is what is happening in the first command i usesd to run the base DP image.

I know this image works, i want to push it to a secure registry
to make it avialabe to other team mates and tools.
I am gon got use the one on IBM cloud. Its basically a private version of Docker Hub.

If I want ot be super cool, I should push that dockerfile and config to GIT. This way many people
can work on it in a team, just like any other cit of code.

docker push...

now I can give people and tools access to it.

END.











ibmuser@localhost ~]$ docker --v
unknown flag: --v
See 'docker --help'.

Usage:	docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default
                           "/home/ibmuser/.docker")
  -c, --context string     Name of the context to use to connect to the
                           daemon (overrides DOCKER_HOST env var and
                           default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level
                           ("debug"|"info"|"warn"|"error"|"fatal")
                           (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default
                           "/home/ibmuser/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default
                           "/home/ibmuser/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default
                           "/home/ibmuser/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  builder     Manage builds
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  engine      Manage the docker engine
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.

[ibmuser@localhost ~]$ dovker -V
bash: dovker: command not found...
[ibmuser@localhost ~]$ docker -V
unknown shorthand flag: 'V' in -V
See 'docker --help'.

Usage:	docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default
                           "/home/ibmuser/.docker")
  -c, --context string     Name of the context to use to connect to the
                           daemon (overrides DOCKER_HOST env var and
                           default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level
                           ("debug"|"info"|"warn"|"error"|"fatal")
                           (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default
                           "/home/ibmuser/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default
                           "/home/ibmuser/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default
                           "/home/ibmuser/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  builder     Manage builds
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  engine      Manage the docker engine
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.

[ibmuser@localhost ~]$ docker -v
Docker version 19.03.7, build 7141c199a2
[ibmuser@localhost ~]$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

[ibmuser@localhost ~]$ ^C
[ibmuser@localhost ~]$ docker -v
Docker version 19.03.7, build 7141c199a2
[ibmuser@localhost ~]$
[ibmuser@localhost ~]$
[ibmuser@localhost ~]$ docker images
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
ibmcom/datapower                                      2018.4.1            4975fdceebe6        2 months ago        873MB
hello-world                                           latest              fce289e99eb9        16 months ago       1.84kB
us.icr.io/chase-registry-namespace/hello-world-repo   first               fce289e99eb9        16 months ago       1.84kB
[ibmuser@localhost ~]$ docker images -a
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
ibmcom/datapower                                      2018.4.1            4975fdceebe6        2 months ago        873MB
hello-world                                           latest              fce289e99eb9        16 months ago       1.84kB
us.icr.io/chase-registry-namespace/hello-world-repo   first               fce289e99eb9        16 months ago       1.84kB
[ibmuser@localhost ~]$ docker rmi -f ^C
[ibmuser@localhost ~]$ docker rmi -f 4975fdceebe6
Error response from daemon: conflict: unable to delete 4975fdceebe6 (cannot be forced) - image is being used by running container 1226d4b9f511
[ibmuser@localhost ~]$ docker ps
CONTAINER ID        IMAGE                       COMMAND             CREATED             STATUS              PORTS                    NAMES
1226d4b9f511        ibmcom/datapower:2018.4.1   "/start.sh"         2 weeks ago         Up 2 weeks          0.0.0.0:9090->9090/tcp   laughing_beaver
[ibmuser@localhost ~]$ docker ps -a
CONTAINER ID        IMAGE                       COMMAND             CREATED             STATUS                      PORTS                    NAMES
3418950a2577        hello-world                 "/hello"            38 minutes ago      Exited (0) 38 minutes ago                            loving_bohr
1226d4b9f511        ibmcom/datapower:2018.4.1   "/start.sh"         2 weeks ago         Up 2 weeks                  0.0.0.0:9090->9090/tcp   laughing_beaver
[ibmuser@localhost ~]$ docker stop 1226d4b9f511
1226d4b9f511
[ibmuser@localhost ~]$ docker container rm

"docker container rm" requires at least 1 argument.
See 'docker container rm --help'.

Usage:  docker container rm [OPTIONS] CONTAINER [CONTAINER...]

Remove one or more containers
[ibmuser@localhost ~]$
[ibmuser@localhost ~]$ docker ps -a
CONTAINER ID        IMAGE                       COMMAND             CREATED             STATUS                         PORTS               NAMES
3418950a2577        hello-world                 "/hello"            About an hour ago   Exited (0) About an hour ago                       loving_bohr
1226d4b9f511        ibmcom/datapower:2018.4.1   "/start.sh"         2 weeks ago         Exited (0) 28 minutes ago                          laughing_beaver
[ibmuser@localhost ~]$ docker container rm 3418950a2577 1226d4b9f511
3418950a2577
1226d4b9f511
[ibmuser@localhost ~]$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[ibmuser@localhost ~]$ docker images -a
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
ibmcom/datapower                                      2018.4.1            4975fdceebe6        2 months ago        873MB
hello-world                                           latest              fce289e99eb9        16 months ago       1.84kB
us.icr.io/chase-registry-namespace/hello-world-repo   first               fce289e99eb9        16 months ago       1.84kB
[ibmuser@localhost ~]$ docker rmi -f fce289e99eb9
Untagged: hello-world:latest
Untagged: hello-world@sha256:f9dfddf63636d84ef479d645ab5885156ae030f611a56f3a7ac7f2fdd86d7e4e
Untagged: hello-world@sha256:fc6a51919cfeb2e6763f62b6d9e8815acbf7cd2e476ea353743570610737b752
Untagged: us.icr.io/chase-registry-namespace/hello-world-repo:first
Untagged: us.icr.io/chase-registry-namespace/hello-world-repo@sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
Deleted: sha256:fce289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e
Deleted: sha256:af0b15c8625bb1938f1d7b17081031f649fd14e6b233688eea3c5483994a66a3
[ibmuser@localhost ~]$ docker images -a
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ibmcom/datapower    2018.4.1            4975fdceebe6        2 months ago        873MB
[ibmuser@localhost ~]$ docker rmi -f 4975fdceebe6
Untagged: ibmcom/datapower:2018.4.1
Untagged: ibmcom/datapower@sha256:58694a971d035f95737c8d91f318767761d463316aa93336922c229f02216b4c
Deleted: sha256:4975fdceebe6881b7ff4a658df8aeba02d4be61b7c57fe4dd1a3a47bca203300
Deleted: sha256:1764444b1abf30f60ed3a015243bcbc06e0c8c651695c110256d4c100c0c26ec
Deleted: sha256:b68d6f44cdd494c9a1c9cd086892ed0770b8b2e677c460c3b40e3145e54046c5
[ibmuser@localhost ~]$ docker images -a
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[ibmuser@localhost ~]$ cd !
bash: cd: !: No such file or directory
[ibmuser@localhost ~]$ cd ~
[ibmuser@localhost ~]$ ls
chaseapikey            Desktop    DPonDocker1   helm                                 Music      Public     Workspace
cluster4cp4ipass       Documents  faspionotes   ibmclouddockerpushtraceandother.txt  oc.tar.gz  Templates
datapowersampleconfig  Downloads  ftptransfers  Komodo-IDE-12                        Pictures   Videos
[ibmuser@localhost ~]$ cd DPonDocker1/
[ibmuser@localhost DPonDocker1]$ ls
config  local
[ibmuser@localhost DPonDocker1]$ cd config/
[ibmuser@localhost config]$ ls
auto-startup.cfg  auto-user.cfg  domains-by-chase
[ibmuser@localhost config]$ git add .
fatal: Not a git repository (or any parent up to mount point /home)
Stopping at filesystem boundary (GIT_DISCOVERY_ACROSS_FILESYSTEM not set).
[ibmuser@localhost config]$ ls
auto-startup.cfg  auto-user.cfg  domains-by-chase
[ibmuser@localhost config]$ cd ..
[ibmuser@localhost DPonDocker1]$ ls
config  local
[ibmuser@localhost DPonDocker1]$ git init
Initialized empty Git repository in /home/ibmuser/DPonDocker1/.git/
[ibmuser@localhost DPonDocker1]$ ls
config  local
[ibmuser@localhost DPonDocker1]$ ls -a
.  ..  config  .git  local
[ibmuser@localhost DPonDocker1]$  git add .
error: open("config/auto-startup.cfg"): Permission denied
error: unable to index file config/auto-startup.cfg
fatal: adding files failed
[ibmuser@localhost DPonDocker1]$ sudo git add .
[sudo] password for ibmuser:
[ibmuser@localhost DPonDocker1]$ git remote add origin horvatca@github.com:horvatca/datapowersampleconfig
[ibmuser@localhost DPonDocker1]$ git remote add origin https://github.com/horvatca/datapowersampleconfig
fatal: remote origin already exists.
[ibmuser@localhost DPonDocker1]$ git push -u origin master
The authenticity of host 'github.com (140.82.114.4)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
RSA key fingerprint is MD5:16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? y
Please type 'yes' or 'no': yes
Warning: Permanently added 'github.com,140.82.114.4' (RSA) to the list of known hosts.
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
[ibmuser@localhost DPonDocker1]$
[ibmuser@localhost DPonDocker1]$
[ibmuser@localhost DPonDocker1]$
[ibmuser@localhost DPonDocker1]$ git remote -v
origin	horvatca@github.com:horvatca/datapowersampleconfig (fetch)
origin	horvatca@github.com:horvatca/datapowersampleconfig (push)
[ibmuser@localhost DPonDocker1]$ git remote rm ^C
[ibmuser@localhost DPonDocker1]$ git remote rm destination
error: Could not remove config section 'remote.destination'
[ibmuser@localhost DPonDocker1]$ git remote rm horvatca@github.com:horvatca/datapowersampleconfig (fetch)
bash: syntax error near unexpected token `('
[ibmuser@localhost DPonDocker1]$ git remote rm horvatca@github.com:horvatca/datapowersampleconfig
error: Could not remove config section 'remote.horvatca@github.com:horvatca/datapowersampleconfig'
[ibmuser@localhost DPonDocker1]$ git remote rm horvatca/datapowersampleconfig
error: Could not remove config section 'remote.horvatca/datapowersampleconfig'
[ibmuser@localhost DPonDocker1]$ git remote rm horvatca@github.com:horvatca/datapowersampleconfig
error: Could not remove config section 'remote.horvatca@github.com:horvatca/datapowersampleconfig'
[ibmuser@localhost DPonDocker1]$ git remote rm horvatca@github.com:horvatca/datapowersampleconfig
error: Could not remove config section 'remote.horvatca@github.com:horvatca/datapowersampleconfig'
[ibmuser@localhost DPonDocker1]$ sudo git remote rm horvatca@github.com:horvatca/datapowersampleconfig
[sudo] password for ibmuser:
error: Could not remove config section 'remote.horvatca@github.com:horvatca/datapowersampleconfig'
[ibmuser@localhost DPonDocker1]$ sudo git remote rm horvatca/datapowersampleconfig
error: Could not remove config section 'remote.horvatca/datapowersampleconfig'
[ibmuser@localhost DPonDocker1]$ sudo git remote rm datapowersampleconfig
error: Could not remove config section 'remote.datapowersampleconfig'
[ibmuser@localhost DPonDocker1]$ git remote remove origin
[ibmuser@localhost DPonDocker1]$ git remote -v
[ibmuser@localhost DPonDocker1]$ git remote add origin https://https://github.com/horvatca/datapowersampleconfig
[ibmuser@localhost DPonDocker1]$ git commit -m 'First things first'
[master (root-commit) 1a51236] First things first
 3 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 config/auto-startup.cfg
 create mode 100644 config/auto-user.cfg
 create mode 100644 config/domains-by-chase/domains-by-chase.cfg
[ibmuser@localhost DPonDocker1]$ git push origin master
fatal: unable to access 'https://https://github.com/horvatca/datapowersampleconfig/': Could not resolve host: https; Unknown error
[ibmuser@localhost DPonDocker1]$ git remote remove origin
[ibmuser@localhost DPonDocker1]$ git remote add origin https://github.com/horvatca/datapowersampleconfig
[ibmuser@localhost DPonDocker1]$ git remote orivin -v
error: Unknown subcommand: orivin
usage: git remote [-v | --verbose]
   or: git remote add [-t <branch>] [-m <master>] [-f] [--tags|--no-tags] [--mirror=<fetch|push>] <name> <url>
   or: git remote rename <old> <new>
   or: git remote remove <name>
   or: git remote set-head <name> (-a | -d | <branch>)
   or: git remote [-v | --verbose] show [-n] <name>
   or: git remote prune [-n | --dry-run] <name>
   or: git remote [-v | --verbose] update [-p | --prune] [(<group> | <remote>)...]
   or: git remote set-branches [--add] <name> <branch>...
   or: git remote set-url [--push] <name> <newurl> [<oldurl>]
   or: git remote set-url --add <name> <newurl>
   or: git remote set-url --delete <name> <url>

    -v, --verbose         be verbose; must be placed before a subcommand

[ibmuser@localhost DPonDocker1]$ git remote -v
origin	https://github.com/horvatca/datapowersampleconfig (fetch)
origin	https://github.com/horvatca/datapowersampleconfig (push)
[ibmuser@localhost DPonDocker1]$ git remote -v
origin	https://github.com/horvatca/datapowersampleconfig (fetch)
origin	https://github.com/horvatca/datapowersampleconfig (push)
[ibmuser@localhost DPonDocker1]$ gitpush origin master
bash: gitpush: command not found...
[ibmuser@localhost DPonDocker1]$ git push origin master
Username for 'https://github.com': horvatca
Password for 'https://horvatca@github.com':
To https://github.com/horvatca/datapowersampleconfig
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://github.com/horvatca/datapowersampleconfig'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first merge the remote changes (e.g.,
hint: 'git pull') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
[ibmuser@localhost DPonDocker1]$
[ibmuser@localhost DPonDocker1]$ git push -u -f origin master
Username for 'https://github.com': horvatca
Password for 'https://horvatca@github.com':
Counting objects: 7, done.
Delta compression using up to 6 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (7/7), 6.38 KiB | 0 bytes/s, done.
Total 7 (delta 0), reused 0 (delta 0)
To https://github.com/horvatca/datapowersampleconfig
 + e761263...1a51236 master -> master (forced update)
Branch master set up to track remote branch master from origin.
[ibmuser@localhost DPonDocker1]$ ls
config  local
[ibmuser@localhost DPonDocker1]$ git add .
[ibmuser@localhost DPonDocker1]$ git commit -m 'First things first hmmm'
# On branch master
nothing to commit, working directory clean
[ibmuser@localhost DPonDocker1]$ ls -a
.  ..  config  .git  local
[ibmuser@localhost DPonDocker1]$ cd git
bash: cd: git: No such file or directory
[ibmuser@localhost DPonDocker1]$ di .git
bash: di: command not found...
Similar command is: 'id'
[ibmuser@localhost DPonDocker1]$ cd .git
[ibmuser@localhost .git]$ ls
branches  COMMIT_EDITMSG  config  description  HEAD  hooks  index  info  logs  objects  refs
[ibmuser@localhost .git]$ cd info
[ibmuser@localhost info]$ ls
exclude
[ibmuser@localhost info]$ cat exclude
# git ls-files --others --exclude-from=.git/info/exclude
# Lines that start with '#' are comments.
# For a project mostly in C, the following would be a good set of
# exclude patterns (uncomment them if you want to use them):
# *.[oa]
# *~
[ibmuser@localhost info]$ cd ..
[ibmuser@localhost .git]$ cd ..
[ibmuser@localhost DPonDocker1]$ ls
config  local
[ibmuser@localhost DPonDocker1]$ git add local
[ibmuser@localhost DPonDocker1]$ git commit -m 'adding local'
# On branch master
nothing to commit, working directory clean
[ibmuser@localhost DPonDocker1]$ git push
warning: push.default is unset; its implicit value is changing in
Git 2.0 from 'matching' to 'simple'. To squelch this message
and maintain the current behavior after the default changes, use:

  git config --global push.default matching

To squelch this message and adopt the new behavior now, use:

  git config --global push.default simple

See 'git help config' and search for 'push.default' for further information.
(the 'simple' mode was introduced in Git 1.7.11. Use the similar mode
'current' instead of 'simple' if you sometimes use older versions of Git)

Username for 'https://github.com': horvatca
Password for 'https://horvatca@github.com':
Everything up-to-date
[ibmuser@localhost DPonDocker1]$ ls
config  local
[ibmuser@localhost DPonDocker1]$ cd local
[ibmuser@localhost local]$ ls
domains-by-chase
[ibmuser@localhost local]$ cat domains-by-chase/
cat: domains-by-chase/: Is a directory
[ibmuser@localhost local]$ cd domains-by-chase/
[ibmuser@localhost domains-by-chase]$ ls
[ibmuser@localhost domains-by-chase]$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[ibmuser@localhost domains-by-chase]$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[ibmuser@localhost domains-by-chase]$     mkdir ~/DPonDocker1
mkdir: cannot create directory ‘/home/ibmuser/DPonDocker1’: File exists
[ibmuser@localhost domains-by-chase]$     cd ~/DPonDocker1
[ibmuser@localhost DPonDocker1]$     docker pull ibmcom/datapower:2018.4.1
2018.4.1: Pulling from ibmcom/datapower
acb220039030: Pull complete
bbde9043ccce: Pull complete
Digest: sha256:58694a971d035f95737c8d91f318767761d463316aa93336922c229f02216b4c
Status: Downloaded newer image for ibmcom/datapower:2018.4.1
docker.io/ibmcom/datapower:2018.4.1
[ibmuser@localhost DPonDocker1]$
[ibmuser@localhost DPonDocker1]$     docker run -it \
>       -v $PWD/config:/drouter/config \
>       -v $PWD/local:/drouter/local \
>       -e DATAPOWER_ACCEPT_LICENSE=true \
>       -e DATAPOWER_INTERACTIVE=true \
>       -e DATAPOWER_WORKER_THREADS=4 \
>       -p 9090:9090 \
>       ibmcom/datapower:2018.4.1
INFO[0000] Starting snmp exporter (version=0.11.0, branch=HEAD, revision=e4591716c29459cb2a12b1bed129af519ad91d23)  source="main.go:138"
INFO[0000] Build context (go=go1.10.2, user=root@80735d30559d, date=20180530-10:24:52)  source="main.go:139"
INFO[0000] Listening on :63512                           source="main.go:218"
20200505T203851.311Z [0x8040006b][system][notice] logging target(default-log): Logging started.
20200505T203851.670Z [0x804000fe][system][notice] : Container instance UUID: 38f0ce3e-b5aa-45fd-88ee-ad1242508f41, Cores: 4, vCPUs: 4, CPU model: Intel(R) Core(TM) i7-6820HQ CPU @ 2.70GHz, Memory: 9628.8MB, Platform: docker, OS: dpos, Edition: developers-limited, Up time: 0 minutes
20200505T203851.790Z [0x8040001c][system][notice] : DataPower IDG is on-line.
20200505T203851.791Z [0x8100006f][system][notice] : Executing default startup configuration.
^C20200505T203852.015Z [0x04f30005][mgmt][error] apic-gw-service(default): tid(975): The effective gateway peering object is down
20200505T203852.016Z [0x04f30005][cli][error] apic-gw-service(default): The effective gateway peering object is down
20200505T203852.016Z [0x00350015][mgmt][notice] api-security-token-manager(default): tid(1039): Operational state down
20200505T203852.017Z [0x8100021f][cli][error] : *** Unknown test "parse-settings"
20200505T203852.146Z [0x8100006d][system][notice] : Executing system configuration.
20200505T203852.148Z [0x8100006b][mgmt][notice] domain(default): tid(8735): Domain operational state is up.
69a70347650e
Unauthorized access prohibited.
20200505T203852.690Z [0x8040009e][system][notice] throttle(Throttler): tid(1647): Setting throttle thresholds: Memory(-1.000000,-1.000000), Temporary-FS(0.000000,0.000000), XML-Names(0.100000), Timeout(30)
20200505T203852.690Z [0x8040009e][system][notice] throttle(Throttler): tid(1647): Setting throttle thresholds: Memory(-1.000000,-1.000000), Temporary-FS(0.000000,0.000000), XML-Names(0.100000), Timeout(30)
20200505T203852.703Z [0x806000dd][system][notice] cert-monitor(Certificate Monitor): tid(415): Enabling Certificate Monitor to scan once every 1 days for soon to expire certificates
20200505T203852.809Z [0x8100006e][system][notice] : Executing startup configuration.
20200505T203852.816Z [0x8040009e][system][notice] throttle(Throttler): tid(1647): Setting throttle thresholds: Memory(-1.000000,-1.000000), Temporary-FS(0.000000,0.000000), XML-Names(0.100000), Timeout(30)
20200505T203852.827Z [0x00350015][mgmt][notice] b2b-persistence(B2BPersistence): tid(111): Operational state down
20200505T203853.786Z [0x00350014][mgmt][notice] quota-enforcement-server(QuotaEnforcementServer): tid(799): Operational state up
20200505T203853.820Z [0x00330091][mgmt][error] source-https(httpshandler1): tid(111): A SSL proxy profile is not configured correctly for this service.
20200505T203853.820Z [0x00330091][cli][error] source-https(httpshandler1): A SSL proxy profile is not configured correctly for this service.
20200505T203853.821Z [0x00330091][mgmt][error] source-https(httpsssssss): tid(111): A SSL proxy profile is not configured correctly for this service.
20200505T203853.821Z [0x00330091][cli][error] source-https(httpsssssss): A SSL proxy profile is not configured correctly for this service.
20200505T203853.826Z [0x04f30005][mgmt][error] apic-gw-service(default): tid(975): The effective gateway peering object is down
20200505T203853.826Z [0x04f30005][mgmt][error] apic-gw-service(default): tid(975): The effective gateway peering object is down
20200505T203853.826Z [0x00350015][mgmt][notice] api-security-token-manager(default): tid(1039): Operational state down
20200505T203853.826Z [0x04f30005][mgmt][error] apic-gw-service(default): tid(975): The effective gateway peering object is down
20200505T203853.826Z [0x04f30005][mgmt][error] apic-gw-service(default): tid(975): The effective gateway peering object is down
20200505T203853.827Z [0x00350015][mgmt][notice] api-security-token-manager(default): tid(1039): Operational state down
20200505T203853.828Z [0x04f30005][mgmt][error] apic-gw-service(default): tid(975): The effective gateway peering object is down
20200505T203853.836Z [0x04f30005][mgmt][error] apic-gw-service(default): tid(975): The effective gateway peering object is down
20200505T203853.836Z [0x04f30005][cli][error] apic-gw-service(default): The effective gateway peering object is down
20200505T203853.839Z [0x00350015][mgmt][notice] smtp-server-connection(default): tid(7407): Operational state down
20200505T203853.840Z [0x00350014][mgmt][notice] smtp-server-connection(default): tid(7407): Operational state up
20200505T203853.858Z [0x00330061][mgmt][error] source-https(httpshandler1): tid(111): A dependent object could not be found
20200505T203853.858Z [0x00330019][mgmt][error] source-https(httpshandler1): tid(111): Operation state transition to up failed
20200505T203853.858Z [0x00350014][mgmt][notice] mpgw(empty-mpgw): tid(111): Operational state up
20200505T203853.859Z [0x80e00386][mpgw][error] mpgw(word): tid(111): Request to activate gateway on 0.0.0.0:443 failed. The conflicting registration is owned by empty-mpgw in domain default.
20200505T203853.859Z [0x80e00116][mpgw][error] mpgw(word): tid(111): httpshandler1 Cannot be started. Already used by another gateway?
20200505T203853.860Z [0x00b30002][mgmt][error] mpgw(word): tid(111): Failed to install on port
20200505T203853.860Z [0x00330019][mgmt][error] mpgw(word): tid(111): Operation state transition to up failed
20200505T203853.868Z [0x00350015][mgmt][notice] quota-enforcement-server(QuotaEnforcementServer): tid(799): Operational state down
20200505T203853.878Z [0x00350014][mgmt][notice] web-mgmt(WebGUI-Settings): tid(303): Operational state up
20200505T203853.883Z [0x8100006b][mgmt][notice] domain(domains-by-chase): tid(28111): Domain operational state is up.
20200505T203853.884Z [0x8100001f][mgmt][notice] domain(domains-by-chase): tid(28111): Created domain folder 'logtemp:'
20200505T203853.885Z [0x8100001f][mgmt][notice] domain(domains-by-chase): tid(28111): Created domain folder 'logstore:'
20200505T203853.885Z [0x8100001f][mgmt][notice] domain(domains-by-chase): tid(28111): Created domain folder 'temporary:'
20200505T203853.885Z [0x8100001f][mgmt][notice] domain(domains-by-chase): tid(28111): Created domain folder 'export:'
20200505T203853.885Z [0x8100001f][mgmt][notice] domain(domains-by-chase): tid(28111): Created domain folder 'cert:'
20200505T203853.885Z [0x8100001f][mgmt][notice] domain(domains-by-chase): tid(28111): Created domain folder 'chkpoints:'
20200505T203853.886Z [0x8100001f][mgmt][notice] domain(domains-by-chase): tid(28111): Created domain folder 'policyframework:'
20200505T203853.886Z [0x8100001f][mgmt][notice] domain(domains-by-chase): tid(28111): Created domain folder 'dpnfsstatic:'
20200505T203853.886Z [0x8100001f][mgmt][notice] domain(domains-by-chase): tid(28111): Created domain folder 'dpnfsauto:'
20200505T203853.886Z [0x8100001f][mgmt][notice] domain(domains-by-chase): tid(28111): Created domain folder 'ftp-response:'
20200505T203853.888Z [0x8100001f][mgmt][notice] domain(domains-by-chase): tid(28111): Created domain folder 'xm70store:'
20200505T203853.894Z [0x8100003b][mgmt][notice] domain(default): Domain configured successfully.
20200505T203854.010Z [domains-by-chase][0x8040006b][system][notice] logging target(default-log): tid(111): Logging started.
20200505T203854.011Z [0x00350014][mgmt][notice] quota-enforcement-server(QuotaEnforcementServer): tid(799): Operational state up
20200505T203854.148Z [domains-by-chase][0x04f30005][mgmt][error] apic-gw-service(default): tid(307): The effective gateway peering object is down
20200505T203854.148Z [domains-by-chase][0x04f30005][cli][error] apic-gw-service(default): The effective gateway peering object is down
20200505T203854.149Z [domains-by-chase][0x00350015][mgmt][notice] api-security-token-manager(default): tid(371): Operational state down
20200505T203854.150Z [domains-by-chase][0x8100021f][cli][error] : tid(28111): *** Unknown test "parse-settings"
20200505T203854.284Z [0x8100003b][mgmt][notice] domain(domains-by-chase): Domain configured successfully.
20200505T203855.792Z [0x804001bb][system][warn] : tid(3): Unable to initialize dpmon process
^C
^C
