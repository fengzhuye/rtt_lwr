Installation instructions
=========================

Requirements
------------

- **Ubuntu 14.04 LTS**
- **ROS Indigo** - Desktop (not desktop-full)  : http://wiki.ros.org/indigo/Installation/Ubuntu
- **OROCOS 2.8** ``(from the ros debian repos if not on xenomai)``
- **Gazebo 7** ``(you can work with older version, but please prefer the latest)``

.. note:: If you are working on Xenomai, you need to compile the OROCOS toolchain from source to build the ``deployer-xenomai`` executable.

.. warning:: This installation assumes you have a freshly installed Ubuntu 14.04 LTS **without** ROS installed. Otherwise, please make sure this does not break any of your packages.

.. note:: We'll use the nice `catkin tools <http://catkin-tools.readthedocs.org/en/latest/>`_ (``catkin build`` instead of ``catkin_make``) to build the packages, it is the recommended way.

.. note::
    * If you're a new user, we recommend to follow the "via debians".
    * If you're a developer, please build everything from source.

ROS Indigo ++
-------------

From  http://wiki.ros.org/indigo/Installation/Ubuntu.

Required tools
~~~~~~~~~~~~~~

.. code-block:: bash

    sudo sh -c "echo 'deb http://packages.ros.org/ros/ubuntu $(lsb_release -cs) main' > /etc/apt/sources.list.d/ros-latest.list"
    wget http://packages.ros.org/ros.key -O - | sudo apt-key add -
    sudo apt update
    sudo apt install python-rosdep python-catkin-tools ros-indigo-catkin python-wstool python-vcstool

Fix Locales
~~~~~~~~~~~~~~

.. code-block:: bash
   
   sudo locale-gen en_US #warnings might occur
   sudo locale-gen en_US-UTF-8
   sudo nano /etc/environment
   # put theses lines
   LANGUAGE=en_US
   LC_ALL=en_US
   # Reboot !
   
If you type ``perl`` you should not see any warnings.

ROS Indigo Desktop
~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    # ROS Desktop (NOT DESKTOP-FULL)
    sudo apt install ros-indigo-desktop

.. warning:: Do not install **desktop-full** (desktop + gazebo 2.2) as we'll use Gazebo 7.

MoveIt! (via debians)
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    # MoveIt!
    sudo apt install ros-indigo-moveit-full

MoveIt! (from source)
~~~~~~~~~~~~~~~~~~~~~

If you need bleeing-edge features of moveit, compile it from source :

.. code-block:: bash

    mkdir -p ~/moveit_ws/src
    cd ~/moveit_ws/src
    wstool init
    wstool merge https://raw.githubusercontent.com/ros-planning/moveit_docs/indigo-devel/moveit.rosinstall
    wstool update -j2
    cd ~/moveit_ws/
    catkin init
    catkin config --install --extend /opt/ros/indigo
    catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release
    catkin build

After Install
~~~~~~~~~~~~~

.. code-block:: bash

    # Load The environment
    source /opt/ros/indigo/setup.bash
    # Update ROSdep (to get dependencies automatically)
    sudo rosdep init
    rosdep update

OROCOS 2.8 + rtt_ros_integration (via debians)
----------------------------------------------

OROCOS toolchain 2.8
~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo apt install ros-indigo-orocos-toolchain ruby1.9.3 ruby-dev libreadline-dev

rtt_ros_integration 2.8
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo apt install ros-indigo-rtt-* ros-indigo-eigen-typekit ros-indigo-kdl-typekit

OROCOS 2.9 + rtt_ros_integration (from source)
----------------------------------------------

You are upgrading from orocos 2.8 :

- If you installed orocos 2.8 from the debians, you need to remove them ``sudo apt remote ros-indigo-orocos-toolchain ros-indigo-rtt-*``.
- If you installed orocos 2.8 from source, they can live side by side in a **different** workspace, but always check ``catkin config`` on you lwr_ws to make sure which workspace you're extending.

OROCOS toolchain 2.9
~~~~~~~~~~~~~~~~~~~~

.. code::

    sudo apt ruby1.9.3 ruby-dev libreadline-dev

.. code-block:: bash

    mkdir -p ~/orocos-2.9_ws/src
    cd ~/orocos-2.9_ws/src
    wstool init
    wstool merge https://raw.githubusercontent.com/kuka-isir/rtt_lwr/rtt_lwr-2.0/lwr_utils/config/orocos_toolchain-2.9.rosinstall
    wstool update -j2
    # Get the latest updates
    cd orocos_toolchain
    git submodule foreach git checkout toolchain-2.9
    git submodule foreach git pull
    # Configure the workspace
    cd ~/orocos-2.9_ws/
    catkin init
    catkin config --install --extend /opt/ros/indigo/
    catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release
    # Build
    catkin build

rtt_ros_integration 2.9
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    mkdir -p ~/rtt_ros-2.9_ws/src
    cd ~/rtt_ros-2.9_ws/src
    wstool init
    wstool merge https://github.com/kuka-isir/rtt_lwr/raw/rtt_lwr-2.0/lwr_utils/config/rtt_ros-2.9.rosinstall
    wstool update -j2
    # Configure the workspace
    cd ~/rtt_ros-2.9_ws/
    catkin init
    catkin config --install --extend ~/orocos-2.9_ws/install
    catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release
    # Build (this can take a while)
    catkin build

Additoonnaly, please make sure that these repos (if you have them) are in the right branches (with fixes for rtt) :

.. code-block:: bash

    roscd rtt_dot_service && git remote set-url origin https://github.com/kuka-isir/rtt_dot_service.git && git pull
    roscd fbsched && git remote set-url origin https://github.com/kuka-isir/fbsched.git && git pull
    roscd conman && git remote set-url origin https://github.com/kuka-isir/conman.git && git pull

Use OROCOS with CORBA
---------------------

In order to use the corba interface (connect multiple deployers together), you'll need to build the orocos_ws and rtt_ros_ws with :

.. code-block:: bash

    catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release -DENABLE_MQ=ON -DENABLE_CORBA=ON -DCORBA_IMPLEMENTATION=OMNIORB

Reference : http://www.orocos.org/stable/documentation/rtt/v2.x/doc-xml/orocos-components-manual.html#orocos-corba

Gazebo 7
--------

From http://gazebosim.org/tutorials?tut=install_ubuntu&cat=install.

.. note:: If you already have gazebo 2.2 installed, please remove it : `sudo apt remove gazebo libgazebo-dev ros-indigo-gazebo-*`

.. code-block:: bash

    # Gazebo 7
    curl -ssL http://get.gazebosim.org | sh
    # The ros packages
    sudo apt install ros-indigo-gazebo7-*

.. note:: Don't forget to put source ``source /usr/share/gazebo/setup.sh`` in your ``~/.bashrc`` or you won't have access to the gazebo plugins (Simulated cameras, lasers, etc).

ROS Control
-----------

Just an extra feature for the whole rtt_lwr package.

.. code-block:: bash

    sudo apt install ros-indigo-ros-control* ros-indigo-control*

RTT LWR packages
----------------

Initialization
~~~~~~~~~~~~~~

First create a workspace for all the packages :

.. code-block:: bash

    mkdir -p ~/lwr_ws/src/


Then you can initialize it :

.. code-block:: bash

    cd ~/lwr_ws/
    catkin init

Download
~~~~~~~~

We use wstool (aka workspace tool) to get all the git repos :

.. code-block:: bash

    cd ~/lwr_ws/src
    # We use wstool to download everything
    wstool init
    # Get rtt_lwr base
    wstool merge https://raw.githubusercontent.com/kuka-isir/rtt_lwr/rtt_lwr-2.0/lwr_utils/config/rtt_lwr.rosinstall
    # Get the extra packages
    wstool merge https://raw.githubusercontent.com/kuka-isir/rtt_lwr/rtt_lwr-2.0/lwr_utils/config/rtt_lwr_extras.rosinstall

    # Download
    wstool update -j2

    # Create some extra ros messages (optional, only for ros control)

    #
    # If you are using the DEBIANS :
    #

    source /opt/ros/indigo/setup.bash

    #
    # Otherwise, if you have built rtt_ros from source
    #

    source ~/rtt_ros-2.9_ws/install/setup.bash


    rosrun rtt_roscomm create_rtt_msgs control_msgs
    rosrun rtt_roscomm create_rtt_msgs controller_manager_msgs

Get the kuka **friComm.h** file (description of the data passing on the ethernet port) :

.. code-block:: bash

    curl https://raw.githubusercontent.com/IDSCETHZurich/re_trajectory-generator/master/kuka_IK/include/friComm.h >> ~/lwr_ws/src/rtt_lwr/lwr_hardware/kuka_lwr_fri/include/kuka_lwr_fri/friComm.h

Check dependencies
~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cd ~/lwr_ws
    rosdep check --from-path src/ -i

Should output :

.. code-block:: bash

    $ System dependencies have not been satisified:
    $ apt gazebo2

Which is **normal** as the default Gazebo version for ros-indigo is **2.2**.
If there are **other** missing dependencies :

.. code-block:: bash

    cd ~/lwr_ws
    rosdep install --from-path src/ -i

Configure the workspace
~~~~~~~~~~~~~~~~~~~~~~~

If using the **debians** :

.. code-block:: bash

    cd ~/lwr_ws
    # Load ROS workspace if not already done
    source /opt/ros/indigo/setup.bash

    catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release

If building rtt_ros **from source** :

.. code-block:: bash

    cd ~/lwr_ws
    catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release --extend ~/rtt_ros-2.9_ws/install

Build the workspace
~~~~~~~~~~~~~~~~~~~

Let's build the entire workspace :

.. code-block:: bash

    cd ~/lwr_ws #it can be anywhere inside the workspace, see catkin_tools docs
    catkin build

.. image:: /_static/catkin-build.png

.. tip::
    To make sure you have the right ROS environnement loaded you can explicitly tell this workspace only needs ROS from debian ``catkin config --extend /opt/ros/indigo``.
    To unset it you can use ``--no-extend``. More info at http://catkin-tools.readthedocs.io/en/latest/verbs/catkin_config.html


Once it's done, load the workspace :

.. code-block:: bash

    source ~/lwr_ws/devel/setup.bash

.. tip:: Put it in you bashrc : ``echo `source ~/lwr_ws/devel/setup.bash` >> ~/.bashrc``

Now we can :doc:`test the installation <test-install>`.
