# How to contribute to the Salzburg Braindynamics Wiki

We always welcome contributions to the Salzburg Braindynamics Wiki from lab members.

## General rules

## Setting up the development environment
We have decided to use the following tools:

1. [Pixi](https://pixi.sh) for dependency management.

### Install the software
The first thing to do is to install [Pixi](https://pixi.sh) which manages all the rest for us.

Check the [installation guide on their website](https://pixi.sh/latest/#installation)

Afterwards, just run `pixi install` in the root folder of the repository and
it's going to take care of the rest.

This is going to create a so-called "environment", i.e. a folder in which all
the dependencies are installed. To use the environment, you need to run `pixi shell`.
