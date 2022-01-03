This project is forked from [Ros-Devcontainer-Upstream](https://github.com/devrt/ros-devcontainer-vscode)

Changes from upstream
---------------------
Changes to `.devcontainer/devcontainer.json`

 1. Set to use ROS vscode extension from microsoft 

Changes to `Dockerfile`

Additional ROS packages installed:

 1. Package 1
 
Configuration changes:
 
 1. Allow specifying ROS version in an `environment` file.

ROS dev container for VSCode
----------------------------
Packed with:
- Preconfigured docker image for ROS development.
- Browser accessible X11 server to display gazebo, rviz, rqt (runs on Windows/Mac).
- Tasks definition to run catkin_make, roscore, rviz commands.
- Preconfigured code completion for C++, Python, XML (package.xml, launchfiles, URDF, SDF).
- Preconfigured simulation environments (Flatland, TurtleBot3, ARIAC, Virtual RobotX, UUV).
- Bonus: WebIDE (Theia) with preconfigured C++, Python, XML completion.

VSCode and devcontainer running on Mac:
![screenshot](https://user-images.githubusercontent.com/18067/58605055-8dc84980-82d1-11e9-8ee5-dc969fcb2ae1.png)

WebIDE (Theia) opened from the local browser while devcontainer is running on the remote server:
![screenshot-theia](https://user-images.githubusercontent.com/18067/59972289-58a8d180-95c7-11e9-86fd-7d271684e8b3.PNG)

How to select simulation environment
-------------------------------------
You can run preconfigured simulation environment as a docker sidecar container.

Enter following command to select the simulator:
```shell
$ ./select-simulator.sh
```

Preconfigured simulation environment currently includes: Flatland, TurtleBot3, ARIAC, Virtual RobotX, UUV.

See the following index for list of current simulators:

https://github.com/devrt/simulator-index/blob/master/index.yaml

If you want any other simulator, let us know by submitting the issue:

https://github.com/devrt/simulator-index/issues

How to use the WebIDE (recommended)
-------------------------------------
As of writing, docker-compose support of VSCode is not so stable on all the platforms.
We recommend using Theia WebIDE since it has complete VSCode function support after 1.0 release.

1. Clone this repository:
```shell
$ git clone https://github.com/devrt/ros-devcontainer-vscode.git
```

2. Enter the following command under the folder of the cloned project:
```shell
$ cd ros-devcontainer-vscode
$ docker-compose up
```

3. Open http://localhost:3001/ using your favorite browser.

You can also use remote server to host the devcontainer (run `docker-compose up` on the remote server and open `http://[remote-server]:3001`).

Eclipse Theia web-prerequisites
-------------------------------

Eclipse Theia Blueprint (desktop)-prerequisites
-----------------------------------------------

VSCode Usage prerequisites
--------------------

  1. Install vscode from official channels or ensure that the vscode version installed is compatible with the remote development extensions.
  0. Install the remote development extensions [Remote-development-main](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack).  This includes the [containers-remote-extension]([https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
  0. Install docker and docker-compose for the platform you're on
     - [Windows](https://docs.docker.com/desktop/windows/install/)
     - [Mac](https://docs.docker.com/desktop/mac/install/)
     - [Docker-on-ubuntu](https://docs.docker.com/engine/install/ubuntu/) and [Compose](https://docs.docker.com/compose/install/).  For compose choose Linux on the install options.  It should be bundled for Windows and Mac.
     - Arch: `sudo pacman -S docker` and `sudo pacman -S docker-compose`.  Then `sudo systemctl start docker` and `sudo systemctl enable docker`.
      - Linux-based systems: ensure that the user is in the docker group so that no extra priviliges are required to run docker.  `sudo usermod -aG docker $USER`.
      - Test with `docker run hello-world`.
  0. Test that the dev-container works:
     - Clone this git repository anywhere.  `git clone git@github.com:jdg-11/ros-devcontainer-vscode.git`
     - Open vscode and type `Ctrl+Shift+p` to bring up the command entry and type `file: open folder` and hit enter.  Open the folder containing the git repository from above that was just checked out.
     - If everything is working, VSCode should detect the `.devcontainer` folder and prompt you to reopen the folder in a container.  Select yes.
      This will build for a brief period to create the docker image and mount it from within VSCode.
      After this, in the bottom left corner should be an indication that one is in a Dev container, and you 
      be able to see the extensions installed e.g. ROS, CMake into VSCode.
 0. Make configuration changes permanent by editing the appropriate files.
      - For additional ROS dependencies edit the `Dockerfile`
      - For additional VSCode extensions edit `.devcontainer/devcontainer.json`
      - For setting up new isolated docker volumes for different projects, edit `docker-compose.yml` 
      - For making sure Catkin and CMake autoloading works, edit ...

### Note:
Sometimes, VSCode may suggest installing CMakeTools.  This is not necessary as the CMakeLists are generated by catkin_create_pkg.

### Note:
Disable telemetry.

% Todo: is it possible to disable all telemetry and phoning services at devcontainer build time?

Quickstart for working with new Catkin workspace (VSCode)
---------------------------------------------------------

Naive attempt:
 1. Start by creating a new directory for the workspace: for example my_catkin_ws.
 2. Enter the directory and clone this git repo: `git clone git@github.com:jdg-11/ros-devcontainer-vscode.git .` (the space '.' is syntactic).
 3. Change the git repository information or `yes | rm -rf .git` and initialize a new git repository if one is needed.
 4. Open vscode and open this folder.  It will autodetect the `.devcontainer` and offer to reopen the folder in a container.

If you do this, you'll see that you're in the same project as before, even though the underlying folder is different.  To test, clone the repo into a different folder, launch another VSCode instance, open the new folder, and reopen in devcontainer.  If you add or change any files, the additions and changes will show up in both running instances of the editor.  

What is happening is that the container in which the devcontainer is running is using a single, virtual, isolated container volume.  The files accessible from within the container do not correspond to local files.  See [Isolated-local-volume](https://code.visualstudio.com/docs/remote/containers#_quick-start-open-a-git-repository-or-github-pr-in-an-isolated-container-volume).  There are advantages to using isolated Docker volumes as opposed to binding to the local filesystem.  These include: ability to move the Docker volume, performance, and maintaining reproducibility (because there's no mutable, local state being passed to the container).

However, one may want to work with different ROS workspaces.  To accomplish this, one just needs to give a new volume name.  In the `docker-compose.yml`, find the definition of the `workspace` service.  Then find the `volumes:` section of the `workspace` service.  One of the volumes is written as `workspace:/workspace`.  In docker a volume is written as `VOLUME_NAME:CONAINER_MOUNT_POINT`.  If the volume name is starts with `/` or `~/` then it is a local binding; otherwise, docker creates an isolated container volume and mounts it to the container mount point specified.  In the example here, `workspace` is telling docker to create an isolated container volume named workspace.  This is what we need to change.  For example, change it to workspace2 as in `workspace2:/workspace`.  The volume workspace is also exposed to it can be used by other containers, and this is declared i nthe `volumes:` description (after the end of the `services:` definition).  At the end of the `docker-compose.yml` find the line 

    volumes:
      workspace:

And change it so that workspace is whatever was chosen above.  In our example, we would change it to 

    volumes:
      workspace2:

Now you can restart code, reopen the folder, and you'll have a fresh dev environment with its own isolated volume.  At this point, you can initialize a git repository from within VSCode running remotely in the container, and setup a catkin workspace.  

As an example workflow, we'll run through a tutorial of using the devcontainer and ROS tool integration in VSCode to build the Message and Server tutorial from (http://wiki.ros.org/ROS/Tutorials) up to running the server and client.

  1. create a working directory `mkdir -p $HOME/Test_Ros_Proj && cd $HOME/Test_Ros_Proj` 
  2. clone the git repository `git clone git@github.com:jdg-11/ros-devcontainer-vscode.git .`
  3. remove the git information `rm -rf .git`
  4. change the volumes to workspace2 by setting `workspace2:/workspace` in the volumes section of the workspace service and setting the `volumes:` to point to `workspace2` at the end of the `docker-compose.yml` file.
  5. start vscode and open the folder and then choose to reopen the folder in container and wait for build to complete
  6. Click the `catkin_make` task button along the bottom of the image of the screen.  This will initialize the catkin workspace, in particular, it creates the `.catkin_workspace` file directive.  You'll have to restart VScode just this once to load the new catkin workspace: close vscode, reopen it, open the folder if it's not still open, and accept the dialog to reopen in container.  *After this initial load, running the `catkin_make` task button will not require restarting the IDE*.
  7. click the `catkin_create_pkg` task button at the bottom or type `Ctrl+shift+p` to open the command prompt and type `ros: create catkin package` and give the package a name. Since we're doing the MsgAndSrv hello-world project we'll call it msg_srv (n.b. lowercase-underscore separate to maintain ROS naming convention).  Next you'll be asked for the dependencies, for the message-server hello world, we use `roscpp rospy std_msgs message_generation`.  This creates the package with the components added to the CMake system.
  8. follow the steps at [Create-Msg-and-Srv](https://wiki.ros.org/ROS/Tutorials/CreatingMsgAndSrv) substituting `msg_srv` as the package name instead of `beginner_tutorials`.  Then follow [Implement-Msg-and-Srv](http://wiki.ros.org/ROS/Tutorials/WritingServiceClient%28c%2B%2B%29) to implement service and client.
  9. Here there are places where we can use the ROS tooling baked into the VSCode instance to make things a bit simpler.  We'll go through the steps below to give a feel for the flow.

Using ROS integration in the VSCode environment (assuming a setup as described above):

  1. The first step is to declare the message and service descriptions to ROS.  While the VSCode tooling doesn't give single click methods for doing this, it does help by providing syntax highlighting for for the special syntax files used in ROS (also this includes .urdf and other files too).
        - After making the package as (step 7) above, rebuild the workspace by clicking the `catkin_make` task button at the bottom of the screen.  From here there are two methods for defining the msg and srv descriptions for the package.
        - The syntactic way is to keep the structure of a catkin project in mind.  A catkin project consists of a folder within a catkin workspace, together with a `CMakeLists.txt` and `package.xml`, and within the folder, a `src` folder to hold source files for the project.  Optionally, within the main package folder are `include`, `msg`, and `srv` folders.  That is, the default configuration expects the `msg` and `srv` declarations for an entire package at that top level.  So you can just mkdir the `msg` and `srv` folders where they're expected.
            
         workspace_name/
            |
            |- .catkin_workspace
            |- src
                |
                |- CMakeLists.txt --global build information
                |- package_name
                         |
                         |- CMakeLists.txt -- package build information
                         |- package.xml  -- build and runtime dependency declarations
                         |- src/  -- specification implementations (.cpp and .h files)
                         |- include/
                         |- msg/  -- message specification (.msg files)
                         |- srv/  -- service specifications (.srv files)
                         |- other-stuff/
       - Alternatively, you can use ROS tooling to tell you where to put things, in particular roscd.  Hit `Ctrl+Shift+p` to bring up the VSCode command dialog, and type `ros: create terminal` and hit enter.  This brings up a bash terminal loaded into the root of the catkin workspace.  Type `source devel/setup.bash` to load the symbols in the project.  Then type `roscd msg_srv`.  This brings you to the correct directory to add the `srv` and `msg` folders.  Type `mkdir msg && mkdir srv`.

  2. Create and edit a file in `msg/Num.msg` containing the single line `int64 num`
  3. Uncomment the lines in the `package.xml` according to [msg-srv](http://wiki.ros.org/ROS/Tutorials/CreatingMsgAndSrv)  section 2.1.
  4. Similarly modify the `CMakeLists.txt` according to [msg-srv](http://wiki.ros.org/ROS/Tutorials/CreatingMsgAndSrv)  section 2.1.
  5. Modify the `CMakeLists.txt` for the `msg_srv` package by removing `message_generation` from the `CATKIN_DEPENDS` in `catkin_package`.  That was autoadded, but we don't need it as message generation happens at build time, and is not needed at runtime.  Alternatively, add `message_generation` as a runtime dependency to `package.xml`.
  6. Create and edit a file `srv/AddTwoInts.srv` containing the following lines

          int64 a
          int64 b
          --- 
          int 64 sum

  7. Uncomment and modify the `package.xml` and `CMakeLists.txt` according to [msg-srv](http://wiki.ros.org/ROS/Tutorials/CreatingMsgAndSrv) section 4.1.
  8. Create and edit a file in the `msg_srv` package's `src` folder `src/add_two_ints_server.cpp` with the contents as decribed by [msg-srv-imp](http://wiki.ros.org/ROS/Tutorials/WritingServiceClient%28c%2B%2B%29) section 1.1. Make sure to replace occurrences of `beginner_tutorials` with `msg_srv`.
  9. Similarly create and edit `src/add_two_ints_client.cpp` with the contents as described by [msg-srv-imp](http://wiki.ros.org/ROS/Tutorials/WritingServiceClient%28c%2B%2B%29) section 2.1.Make sure to replace occurrences of `beginner_tutorials` with `msg_srv`.
  10. Modify the `CMakeLists.txt` for the `msg_srv` package by adding the executable information as described [msg-srv-imp](http://wiki.ros.org/ROS/Tutorials/WritingServiceClient%28c%2B%2B%29) section 3.  Make sure to replace occurrences of `beginner_tutorials` with `msg_srv`.
  11. Click the `roscore` task button at the bottom of the screen or `Ctrl+shift+p` and then `ros: start`.  If you choose the latter, `roscore` will be silently started in the background.  If you choose the former, `roscore` will be started in the background, and it's output will be viewable in a console widget available from the bottom right of the screen.  Either way, the status can be checked with `Ctrl+Shift+p` and `ros: status`.
  12. Click the `catkin_make` task button at the bottom of the screen to rebuild the workspace.
  13. As a first run, type `Ctrl+Shift+p` and then `ros: create terminal`.  This presents a terminal loaded into the root of the catkin workspace.  However, it does not have catkin workspace sourced (this can be evidenced by running `echo $ROS_PACKAGE_PATH`).  Source the environment: by typing `source devel/setup.bash` in the terminal.  Then type `rosrun msg_srv add_two_ints_server`.  This runs the server in the current terminal.
  14. Create a new ros terminal and source environment as in the previous step.  Then type `rosrun msg_srv add_two_ints_client 5 14`.  You should see the client respond with `Sum: 19`.  
  15. In the terminal view in VSCode you can see all running ROS terminals on the right hand side.  You can select them to view output on other terminals.  If you choose the terminal that is running the `add_two_ints_server` you should see a line with `request: x=5, y=14` and then another line with `sending back response: [19]`.  

Finally, if desired, initialize a git_repository in the root of the catkin-workspace from within the VSCode devcontainer.  By creating the git repository within the devcontainer, you'll not pollute the repository with devcontainer information, and it frees up concerns about statefully storing information in a local file system which would break reproducible builds.

There are different options for doing this.  Start by creating a `.gitignore` and blocking at lest the folder `.vscode`, `build` and `devel`.  Then create a git repository using the git tools on the left of vscode, and the appropriate files.  

You can then choose to use the github integration in vscode to publish the repository to github directly.

Another option is to manually set the remote origin in the VSCode terminal from within the devcontainer.  To push the project to a git repo, with gitlab, github, bitbucket, etc, you'd have to make the project there and then push the changes from local.  The devcontainer uses the official Microsoft VSCode method of sharing git credentials with a devcontainer found [git-cred-for-devcontainer](https://code.visualstudio.com/docs/remote/containers#_sharing-git-credentials-with-your-container).  So as long as the host is set up to use ssh keys, the devcontainer will be too.

Then you can manage the git repository using built-in git repos in VSCode.




<!-- Quickstart for working with new Catkin workspace (Theia [web])
--------------------------------------------------------------
TBD -->

<!-- Quickstart for working with new Catkin workspace (Theia [desktop])
------------------------------------------------------------------
TBD -->

<!-- Quickstart for working with new Catkin workspace (Other)
--------------------------------------------------------
TBD -->

Quickstart for working with existing Catkin workspace (VSCode - files in a revision system repo)
------------------------------------------------------------------------------------------------
Here we will clone the git repo created in the Quickstart for new Catkin workspace into a fresh devontainer.


Prepare a fresh volume for the project.
  1. create a working directory `mkdir -p $HOME/proj_workspace && cd $HOME/proj_workspace` 
  2. clone the git repository `git clone git@github.com:jdg-11/ros-devcontainer-vscode.git .`
  3. remove the git information `rm -rf .git`
  4. change the volume to a fresh name: `fresh_volume:/workspace` in the volumes section of the workspace service and setting the `volumes:` to point to `fresh_volume` at the end of the `docker-compose.yml` file.
  5. start vscode and open the folder and then choose to reopen the folder in container and wait for build to complete

Set up any dependencies in the Docker image
  1. Edit Dockerfile and add entries either via `apt` or a ROS direct install method.

Import the git repo into the devcontainer.
  1. Remove the autogenerated `src/` and `compile_flags.txt`.
  2. Click the git management button on the left hand side of vscode (a little branching graph icon. 
  3. Choose initialize git repository
  4. Find the ellipses in the `Source Control` menu at the top of the sidepane.  Select `Remote` and then `Add Remote...` and enter the ssh git repo.
  5. In the `Source Control` menu select `Pull,Push` and then `Pull-from`.  A drop down will show the main remote workspace, and choose the `main` branch to pull from.  
   
At this point, you have cloned the git repository into the devcontainer, and changes made can be tracked using VSCode's builtin git tools.



Quickstart for working with existing Catkin workspace (VSCode - files only local)
---------------------------------------------------------------------------------
Use a host-binding for the workspace by editing the docker-compose.yml.  However, note this is statefully dangerous as it breaks the separation of the dev environment and the build environment.

<!-- Quickstart for working with existing Catkin workspace (Theia [web])
-------------------------------------------------------------------
TBD -->

<!-- Quickstart for working with existing Catkin workspace (Theia [desktop])
-----------------------------------------------------------------------
TBD -->

<!-- Quickstart for working with existing Catkin workspace (Other)
-------------------------------------------------------------
TBD -->



Using templates
---------------

If you are behind the proxy
-----------------------------

Please apply following two settings, if you are using your PC behind the proxy.

1. Proxy setting for the Docker server.

Click Docker Desktop task bar icon > Select `preference` menu item. You will see the following options:

![docker-proxy-settings](https://user-images.githubusercontent.com/18067/59744551-c4302d80-92ad-11e9-9b20-cc873a53a8bb.png)

In most cases, `System proxy` option will work. But if you have problem downloading the docker images, please try `Manual proxy configuration` option.

2. Proxy setting for the devcontainer.

This setting will enable you to use the `apt-get` or the other network commands inside the devcontainer.

First, open `.env.sample` file under the root folder of the cloned project.
Edit the settings according to your environment.
Save the file as name `.env`.

Next, open `docker-compose.yml` file under the root folder of the cloned project and uncomment the following lines:
```yaml
  workspace:
    env_file:
      - .env
```

How to reset or update the devcontainer
---------------------------------------

If you want to reset the devcontainer. Please close vscode and enter the following command under the folder of the cloned project:
```shell
$ docker-compose down
```

If you want to update the environment to the most recent version. Please enter the following commands under the folder of the cloned project:
```shell
$ git pull origin master
$ docker-compose pull
```

Please be noticed that the `docker-compose down` command will reset your environment including installed `.deb` packages. However, if you write `package.xml` files correctly, you can reinstall all the depending packages by entering the following two commands:
```shell
$ rosdep update
$ rosdep install --from-paths src --ignore-src -r -y
```

How to open X11 server screen
-----------------------------

1. Wait for the container to start.
2. Open http://localhost:3000/ using your favorite browser.

If you are using Docker Toolbox, open the following URL instead:

http://192.168.99.100:3000/

If you want browser screen to be integrated with VS Code, use [Browser Preview for VS Code extension](https://marketplace.visualstudio.com/items?itemName=auchenberg.vscode-browser-preview).

Created by
----------
Yosuke Matsusaka (MID Academic Promotions, Inc.)

License
-------
Code in this repository (Dockerfile, utility scripts, etc) is distributed under Apache 2.0 license.

Included components are distributed under each different licenses:
- Theia IDE: EPL or GPL
- Jupyter notebook: BSD
