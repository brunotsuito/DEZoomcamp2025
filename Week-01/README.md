# Week 1 Homework

## Question 1:
Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash.

What's the version of pip in the image?

    24.3.1
    24.2.1
    23.3.1
    23.2.1

### Solution

#### Step 1: Run Docker with the Python Image
I ran the following command to start a Docker container with the `python:3.12.8` image in interactive mode and dropped into a `bash` shell:

docker run -it python:3.12.8 bash

#### Step 2: Checking pip version
Inside the container, I ran the following command to check the pip version:

pip --version

#### The Output was:
pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)

So the correct pip version in the python:3.12.8 image is 24.3.1.




