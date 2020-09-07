# buildroot-ci-loop-demo 

A ready to run CI infrastructure with LAVA for automated testing.

- [Introduction](#Introduction)
- [Install](#Install)
- [Run](#Run)
- [Play](#Play)
- [Customize](#Customize)

## Introduction

Continuous integration (CI) is a development practice by which an in-development project is continuously (or frequently) built and tested to ensure that nothing is broken. Usually, each contribution to the project (e.g. a merge request) is subject to that automated procedure, allowing to detect any new bugs or regressions early, and report them back to the developers, forming a CI loop.

![image](https://people.linaro.org/~loic.poulain/lava/ci-loop.png)

This project is a preconfigured self-contained continuous integration loop based on Git, Jenkins, LAVA and SQUAD components. This preconfigured instance hosts, generates and tests a complete embedded system distro (buildroot based), demoing a real example of LAVA configuration and usage inside a CI loop.

Components:

- **Git**: Version control system, hosting the software (here buildroot)
- **Jenkins**: Automation server for automate building of the software
- **fileserver**: Store the build artifacts (e.g. kernel, rootfs...)
- **LAVA**: Automated testing service, for testing software on real hardware (here we test with qemu devices)
- **SQUAD**: Software quality dashboard for tracking software coverage, regressions...

![image](https://people.linaro.org/~loic.poulain/lava/cibox_loop.png)

This preconfigured CI loop track a **custom buildroot git repository** which contains the buildroot system (git submodule), as well as a buildroot configuration and overlays. This repository is a typical way to create a **custom system** for an **embedded device**. A developer can **fetch** that repository and **push** its own changes. When a new **commit** is pushed to the master branch, jenkins detects it and **build** the system automatically, following the steps defined in the jenkins buildroot job configuration. Once the 'system' blobs have been generated (kernel, rootfs), Jenkins pushes them on the **artifact server** (via ftp) and **triggers testing** (LAVA jobs). LAVA takes care of **automated testing** of the newly generated system on the **device(s)** (in our case, qemu devices) and submit **test results** (pass/fail) to the **quality dashboard** (SQUAD) that users or developers can access to follow software progress.

## Install

Your distro must have **git**, **docker** and **docker-compose** packages installed.

Then clone the project:

    git clone https://github.com/ci-box/buildroot-ci-loop-demo.git
    cd buildroot-ci-loop-demo
    git submodule update --init

And build the instance:
    
    docker-compose build
`Note`: First build needs to download and create docker images, which takes some times... (~20/30min)

For upcoming opertations you need to copy your ssh public key to the gitserver overlay:

    cp ~/.ssh/id_rsa.pub overlays/gitserver/pubkeys/
 (how to generate ssh key: https://git-scm.com/book/en/v2/Git-on-the-Server-Generating-Your-SSH-Public-Key)

## Run

    docker-compose up
`Note`: If you get a permission issue, either use sudo or add your user to the docker group

## Play

#### 0. Info
Once all the services are up and running, you can access them. All are exposed to the network at different ports (listed below). If you want to access them directly from the machine you're currently running the services, then use the localhost addess, either use the machine ip address to access them remotely.

Default addresses are:

| Service | External Port | local address | extra info |
| ------ | ------ | ------ | ------ |
| SQUAD web interface | 8000 | http://localhost:8000 | not need to log in
| Jenkins web interface | 8001 | http://localhost:8001 | login: admin/password
| fileserver web interface | 8002 | http://localhost:8002 ||
| LAVA web interface | 8003 | http://localhost:8003 | login: admin/password
| Git server web interface | 8004 | http://localhost:8004 ||
| Git SSH interface | 8022 | ssh://git@localhost:8022 | ssh pub keys into *overlays/gitserver/pubkeys* |

#### 1. Fetch, modify and push (feed the CI pipeline)

A buildroot git repository has been created by default, you can check that by accessing http://localhost:8023/?p=buildroot.git, as a developer you want to download that source code on your host.

    git clone ssh://git@localhost:8022/git-server/repos/buildroot.git
    cd buildroot

You can read the README file to know how to use and build that software, but here we are just going to perform a simple update, adding additional ramsmp package tool to arm64 config:

    echo "BR2_PACKAGE_RAMSMP=y" >> buildroot/arm64/.config

Then commit and push your change to the git server:

    git commit -a -s -m "test commit"
    git push

`Note`: you can check your change now appears in the git server web interface.

#### 2. Check Jenkins

Since you modified the software, jenkins should automatically run a job to build the software. Connect to the Jenkins web interface (http://localhost:8001), click on buildroot and check that build history shows an ongoing job. You can click on that build number and follow the console output. First build will be quite long since it needs to fetch and build all the system, including the linux kernel.

#### 3. Check artifacts on the fileserver

Once build is complete, jenkins will upload the generated rootfs and kernel to the fileserver in the buildroot directory: http://localhost:8002/buildroot/.

#### 4. Check LAVA jobs

Jenkins is also configured to triggers different test jobs in LAVA against the newly generated system. You can access LAVA jobs from its web interface: http://localhost:8003/scheduler/alljobs.

#### 5. Retrieve results

Access the buildroot project from software quality dashboard (SQUAD) to get information about tested software: http://localhost:8000/Linaro/buildroot/. In this setup, builds are organized per version of the software (derived from commit id). You should see at least one build reported and the associated results (when LAVA finished testing).

#### 6. Continue...

If you push another change to the buildroot repository, jenkins will trigger an other build, testing, report... You can compare builds from the squad interfaces. Your CI loop is operational.

## Customize

#### Docker overlays
#### Adding a test
#### Testing with real device
