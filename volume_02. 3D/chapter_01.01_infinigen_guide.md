# Chapter 01 - 01. Infinigen Getting Started

## 1. Objectives

[Infinigen](https://github.com/princeton-vl/infinigen) is a python library
that utilizes the [Blender python api](https://docs.blender.org/api/current/info_quickstart.html)
to generate synthetic images of nature scene, as well as indoors layouts with furnitures. 

This article is a quickstart guide to use infinigen. 

&nbsp;
## 2. Installation

We installed infinigen in a Razor notebook `RZ09-0421`, empowered with nvidia `geforce RTX 3070Ti` GPU core. 

And our OS is `ubuntu 22.04`. 

Following [the infinigen's guide on installation](https://github.com/princeton-vl/infinigen/blob/main/docs/Installation.md#installing-infinigen-as-a-python-module), 
we installed `the infinigen as a python module` successfully. 

After then we added `infinigen as a blender python script` by executing `interactive_blender.sh`,

~~~
robot@robot-test:~/infinigen$ conda activate infinigen
(infinigen) robot@robot-test:~/infinigen$ pwd
/home/robot/infinigen

(infinigen) robot@robot-test:~/infinigen$ bash scripts/install/interactive_blender.sh
~~~

Hence, we run double modes of infinigen on the same notebook. 

&nbsp;
## 3. Test run

We generated a natural scene in two ways, both were succussful. 

1. The first one is [a step-by-step process](https://github.com/princeton-vl/infinigen/blob/main/docs/HelloWorld.md#generate-a-scene-step-by-step),
   the purpose is to test run `generate_nature.py`.

2. The second one is [one command execution](https://github.com/princeton-vl/infinigen/blob/main/docs/HelloWorld.md#generate-scenes-in-one-command),
   the purpose is to test run `manage_jobs.py`.

The result consists of many files, including many images. The human readable image is in the `outputs/hello_world/0/frames/Image/camera_0/` directory, 

   <p align="center">
     <img alt="Generated RGP image of infinigen" src="./assets/0101_helloworld_desert.png" width="85%">
   </p>

Later on we will dive into the source codes of these two scripts, to study the usage of infinigen in depth. 

