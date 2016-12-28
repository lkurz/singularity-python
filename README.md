# Singularity Python

[![Build Status](https://travis-ci.org/singularityware/singularity-python.svg?branch=master)](https://travis-ci.org/singularityware/singularity-python)

This is a python command line tool for working with [Singularity](singularityware.github.io) containers, specifically providing functions to visualize, package, and containers. Currently, most functions use Singularity on the commnd line, and require sudo, and this is expected to change when the software is updated to obtain image folders and files without an export that uses `Singularity export`.

The Singularity-Python code is licensed under the MIT open source license, which is a highly permissive license that places few limits upon reuse. This ensures that the code will be usable by the greatest number of researchers, in both academia and industry. 


## Installation (current)

      pip install singularity


## Installation (dev)

      pip install git+git://github.com/singularityware/singularity-python.git


## Quick Start

Installation will place an executable, `shub` in your bin folder. 

### Test your installation

After installation, you should be able to run `shub` on the command line, without any input args, to see if you have Singularity installed (and test the package):


      $ shub
	Please specify one or more containers with --image(s)


      $  shub --help
	usage: shub [-h] [--image IMAGE] [--images IMAGES] [--debug]
		    [--outfolder OUTFOLDER] [--package] [--tree] [--simtree]
		    [--subtract] [--simcalc] [--size SIZE]

	Singularity Hub command line tool

	optional arguments:
	  -h, --help            show this help message and exit
	  --image IMAGE         full path to singularity image (for use with --package
		                and --tree)
	  --images IMAGES       images, separated by commas (for use with --simtree)
	  --debug               use verbose logging to debug.
	  --outfolder OUTFOLDER
		                full path to folder for output, stays in tmp (or pwd)
		                if not specified
	  --package             package a singularity container for singularity hub
	  --tree                view the guts of an singularity image (use --image)
	  --simtree             view common guts between two images (use --images)
	  --subtract            subtract one container image from the second to make
                                a difference tree (use --images first,subtract)
	  --simcalc             calculate similarity (number) between images based on
		                file contents.
	  --size SIZE           If using Docker or shub image, you can change size
		                (default is 1024)



### Package your container

A package is a zipped up file that contains the image, the singularity runscript as `runscript`, a `VERSION` file, and a list of files `files.txt` and folders `folders.txt` in the container. 

![img/singularity-package.png](img/singularity-package.png)

The example package can be [downloaded for inspection](http://www.vbmis.com/bmi/project/singularity/package_image/ubuntu:latest-2016-04-06.img.zip), as can the [image used to create it](http://www.vbmis.com/bmi/project/singularity/package_image/ubuntu:latest-2016-04-06.img). This is one of the drivers underlying [singularity hub](http://www.singularity-hub.org) (under development).

  - **files.txt** and **folders.txt**: are simple text file lists with paths in the container, and this choice is currently done to provide the rawest form of the container contents. These files also are used to generate interactive visualizations, and calculate similarity between containers.
  - **VERSION**: is a text file with one line, an md5 hash generated for the image when it was packaged. 
  - **{{image}}.img**: is of course the original singularity container (usually a .img file)

First, go to where you have some images:

      ls
      ubuntu.img
      

You can now use the `shub` command line tool to package your image. Note that you must have [singularity installed](https://singularityware.lbl.gov/install-linux), and depending on the function you use, you will likely need to use sudo. We can use the `--package` argument to package our image:

      shub --image ubuntu.img --package


If no output folder is specified, the resulting image (named in the format `ubuntu.img.zip` will be output in the present working directory. You can also specify an output folder:

      shub --image ubuntu.img --package --outfolder /tmp

For the package command, you will need to put in your password to grant sudo priviledges, as packaging requires using the singularity `export` functionality.

For more details, and a walkthrough with sample data, please see [examples/package_image](examples/package_image)


### View the inside of a container
What's inside that container? Right now, the main way to answer this question is to do some equivalent of ssh. shub provides a command line function for rendering a view to (immediately) show the contents of an image (folders and files) in your web browser. **Important** the browser will open, but you will need to use your password to use Singularity on the command line:


#### Docker Image
shub will render the guts of any Docker image, even if it's not currently on your system. You don't need 


      shub --image docker://ubuntu:latest --tree


#### Singularity Package or Image

      shub --image ubuntu.img --tree


This will open up something that looks like this:

![img/files.png](img/files.png)
An [interactive demo](https://singularityware.github.io/singularity-python/examples/container_tree) is also available, and see the [example](examples/container_tree) for updates.


### Visualize Containers

#### Container Similarity Tree

![examples/similar_tree/simtree.png](examples/similar_tree/simtree.png)

What do two containers have in common, in terms of files and folders? shub provides a command line function for rendering a view to (immediately) show the similarity between to container images:


      shub --images centos.img,ubuntu.img --simtree --debug

      
We can also compare a local image to a Docker image:


      shub --images docker://ubuntu:latest,/home/vanessa/Desktop/ubuntu.img --simtree --debug


Or two Docker images:


      shub --images docker://ubuntu:latest,docker://centos:latest --simtree


If you need output for any of the following, you can add the `--debug` argument. Note that when generating docker comparisons, the back end is obtaining the layers, creating the images, importing and packaging, so the result is not instantanous.


An [interactive demo](https://singularityware.github.io/singularity-python/examples/similar_tree/) is also available.


#### Container Difference Tree
What files and folders differ between two containers? What does it look like if I subtract one image from the second? `shub` provides a command line tool to generate a visualization to do exactly this.


      shub --subtract --images docker://ubuntu:latest.docker://centos:latest

As with `simtree`, this function supports both docker and singularity images as inputs.

![examples/difference_tree/difftree.png](examples/difference_tree/difftree.png)

An [interactive demo](https://singularityware.github.io/singularity-python/examples/difference_tree/) is also available.


### Compare Containers
The same functions above can be used to show the exact similarities (intersect) and differences (files and/or folders unique to two images) between two images. You can get a data structure with this information as follows:


      from shub.views import compare_containers
  
      image1 = 'ubuntu.img'
      image2 = 'centos.img'
      by = "files.txt" # can also be "folders.txt", or a list with both

      comparison = compare_containers(image1,image2,by=by)



#### Calculate similarity of images

We can calculate similarity of images based on the file content inside. For an example, see [examples/calculate_similarity](examples/calculate_similarity). We can compare two local images as follows:

      $ shub --images /home/vanessa/Desktop/ubuntu.img,/home/vanessa/Desktop/ubuntu.img --simcalc
      
and the same applies for specification of Docker images, as in the previous example. Note that we are specifying `images` for the argument instead of `image`, and it's a single string of image names separated by a comma. 



### Build your container
More information coming soon.


### Functions Provided
You can also use the library as a module, and import singularity-python functions into your application. If you would like to see specific examples for something, [please ask](https://github.com/singularityware/singularity-python)!


## Help and Contribution
Please contribute to the package, or post feedback and questions as [issues](https://github.com/singularityware/singularity-python). For points that require discussion of the larger group, please use the [Singularity List](https://groups.google.com/a/lbl.gov/forum/#!forum/singularity).
