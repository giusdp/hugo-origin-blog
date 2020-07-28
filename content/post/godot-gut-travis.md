---
author: "Giuseppe De Palma"
title:  "Automate Godot testing with Docker & Travis"
description: "Containerize your Godot game and run your GUT tests on Travis automatically."
categories: ["Game Dev"]
date: 2020-07-28T11:55:21+00:00 
draft: false
tags: ["game dev"]
imagelink: "Godot/godot-gut-travis.png"
---

Working on my thesis about cloud technologies, I started experimenting with different CI/CD solutions and tried my hand
at setting up a workflow for game development with Godot. I found the best solutions are [GitLab CI/CD](https://about.gitlab.com/product/continuous-integration) and [Travis CI](https://travis-ci.com), but decided for the second one since I mainly use GitHub and the GitLab offer doesn't integrate as well as Travis.

If you're interested on setting up your own automatic testing system for Godot with GUT (Godot Unit Tests), you can use the config files in this article.

An updated empty project already setup with the Dockerfile and the Travis config file is available at this [GitHub repo](https://github.com/giusdp/godot-gut-travis-example).

## Let's start with Docker

Docker is a tool to deploy and run applications inside isolated environments, called containers. You don't need to be an expert or even install it to use it. You just have to put a ``Dockerfile`` inside your project and Travis will take care of it. 

A Dockerfile is used to describe an isolated environment, so that an application can be ran in it without touching the host machine. We will specify an Ubuntu environment with the headless Godot engine and our project inside. Travis will pick up this file, build and run our container and run our tests from inside it.

#### Dockerfile


```docker
FROM ubuntu:bionic

ENV VERSION "3.2"

COPY . ./project

RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates unzip wget \
    && wget https://downloads.tuxfamily.org/godotengine/${VERSION}/Godot_v${VERSION}-stable_linux_headless.64.zip \
    && unzip Godot_v${VERSION}-stable_linux_headless.64.zip \
    && mv Godot_v${VERSION}-stable_linux_headless.64 /usr/local/bin/godot \
    && rm -f Godot_v${VERSION}-stable_linux_headless.64.zip \
    && rm -rf /var/lib/apt/lists/*
```
Create a file, at the root folder of your Godot project, named `Dockerfile` (just that, no file format) and paste in the text.


When your repository receives a commit or a pull request, Travis will notice the new changes and run a new Docker container. To do so, it will find our Dockerfile and will run line by line our instructions. The first line `FROM ubuntu:bionic` will create the basis of our environment, an Ubuntu container. Then we specify the Godot's version we need with the variable `VERSION`, which is used later to complete the url used to download Godot. You can change this string with whatever version you need, just be sure the url will work. 

The `COPY` command is used to copy the content of a folder to inside the container. The `.` means copy the contents of the folder where the Dockerfile is located (which is the entire project folder), and put it in a folder called `project` inside the container. We want to put the game project in its own folder in the container, so Travis knows where to find it.

Lastly, the `RUN` command runs some instructions before having the container ready. We install the minimum necessary to download the headless Godot engine, extract it, rename it `godot` so that it can be used easily in the console and delete leftover files.

## Tests and Travis

Travis CI is a continuous integration service used to build and test projects hosted at GitHub. The idea is that when you push some
changes to your project, Travis will run your tests and notify you if they pass or fail. For a Godot project, the [GUT addon](https://github.com/bitwes/Gut) provides a nice unit testing framework to create tests and run them.

Add GUT to your project. Open the Asset Lib tab, search "GUT" download it and install it. The addons folder with the plugin inside will be placed in the project file system.  

<p align="center">
<img src="/Godot/GUTInstallation.png" alt="Download and install GUT to add the addons folder."/>
</p>

Remember that in the docker container we copy all of the project content, so the addons too. GUT provides a script "gut_cmdln.gd" which can be used to run tests from a command line interface, that is what Travis will use.

Let's add an example test to try things out. Of course you'll have to write real tests while working on the project, or all of this will be useless!

Add a "test" folder at the root level, with a file in it: "test_example.gd".

<p align="center">
  <img src="/Godot/test-folder.png" alt="show test folder image"/>
<p>

Every scripts used for testing have to be put in this folder. I'm not sure if it's required to start their name with "test_", but I think it's a good convention. 

Now we can write the simplest and most useless test there is:
```gdscript
extends "res://addons/gut/test.gd"

func test_assert_true_with_true():
  assert_true(true, "Should pass, true is true")
```

Every test script ***must*** extend the `gut/test.gd` class. Each unit test is a function, but a GUT tutorial is out of the scope of this article. You can find everything on the [GUT repository wiki](https://github.com/bitwes/Gut/wiki).

I don't need to tell what that test does. It always succeeds.

Now let's configure Travis to run our test!

### .travis.yml

Add to your project a new file: `.travis.yml`

```yaml
language: minimal

services:
  - docker

branches:
  only:
    - master

before_install:
  docker build -t project/docker .

script:
  docker run project/docker godot --path project \
   -s addons/gut/gut_cmdln.gd -gdir=res://test -gexit
```

That's it! We set *language* to minimal because we don't need Travis to load any particular programming language. With the *branches* key we specify to only run Travis when a change is made on the master branch. When this happens Travis will first build the docker container and then run it with the instruction at *script*. It runs `gut_cmdln.gd` on the *test* folder.

On the GitHub repository itself it will be shown if the tests pass or fail after a commit or pull request.

Now let's actually connect Travis to our repository so that it can pick up our changes.

## Connect Travis

Head over to [travis-ci.com](https://travis-ci.com) and sign in with GitHub. Once that is done, on the left hit the **+** button. 

<p align="center">
<img src="/Godot/AddTravis.png"/>
</p>

It will take you on the page to add repositories. Do more connecting with GitHub stuff, either give it access to all repos or just the ones you want. I chose the example repo I have.
<p align="center">
<img src="/Godot/SelectRepo.png"/>
</p>

And it's **done**!

Try pushing some changes to the master branch and you'll see the magic happen. On the Travis page of your repo you will have many information about the build and test process. On Github you will see some thing like this:

<p align="center">
<img src="/Godot/PassedTest.png" alt="Test passes"/> or <img src="/Godot/FailedTest.png" alt="Test fails" />
</p>

For the first I pushed the test we wrote that checks if `true` is `true`. For the second I changed it to check if `false` is `true`, to make it fail. Everything works.

Now you can explore the magic of continuous integration with Travis on your Godot projects.
