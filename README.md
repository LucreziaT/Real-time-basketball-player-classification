# Real-time-basketball-player-classification
A real time basketball player classification code written in python3 and based on opencv, Darknet and YOLO.

# Project description
The project goal is to rank the players according to the color of the shirt, including referees.
There are two main types of code, one uses CPU (slower) and the other the Nvidia GPU (faster).
Unfortunately, I was unable to upload the video used to try and develop the code because it is under copyright
# Dependency installation
All the procedure refers to Manjaro Linux but for Ubuntu or other distros it's similar.
```sh
$ cd ~/Document
$ git clone https://github.com/lucacaronti/Real-time-basketball-player-classification.git
$ sudo pacman -S opencv base-devel opencv-samples hdf5 cuda cudnn
$ pip3 install scipy matplotlib numpy imutils --user
$ git clone https://github.com/pjreddie/darknet
$ cd darknet
```
Now we need to change the makefile inside the darknet folder to allow the compilation with cuda and cuda neural network. So set
```makefile
GPU=1
CUDNN=1
```
And in my case I had to change some other path:
```makefile
NVCC=/./opt/cuda/bin/nvcc
COMMON+= -DGPU -I/opt/cuda/include/
LDFLAGS+= -L/opt/cuda/lib64 -lcuda -lcudart -lcublas -lcurand
```
At this point, we must start the compilation.
```sh
$ make -j$(nproc)
```
Now copy the shared object inside our folder
```sh
$ cp ~/Documents/darknet/libdarknet.so ~/Documents/Real-time-basketball-player-classification/code/
```
It's also necessary to download weight for the neural network
```sh
$ wget https://pjreddie.com/media/files/yolov3.weights ~/Documents/Real-time-basketball-player-classification/code/yolo-coco/
```
# Code usage
---
#### videoAverage
It's a script that does the average of frames to eliminate moving objects and get the background (e.g. basketball court). This only works if the shot is taken from a single point without moving the camera.
```sh
$ python videoAverage.py --input ../video/your_video.avi --output ../image/im_out.png
```

---
#### GPU_classifier
It's the code which classifies players using the GPU.
Inside the code there is a variable called **learningFrame** that indicates how many frames must be used to create the initial dataset of player histograms. For each of these frames will compare the detection and program will ask you what kind of player is (player 1, player 2, referee, nothing). At each detection the histogram dataset will upgrade by yourself in order to improve over time.
Usage:
```sh
$ python GPU_classifier.py --input ../video/your_video.avi --output ../video/output.avi --background ../image/background.png
```
The **background** option is optional, if no image is given the background subtraction is not done, which leads to a decrease in precision.

###### How to increase deep neural network precision
Inside the file **yolov3_GPU** there are two variable called **width** and **height**. Higher these variables are, higher is the accuracy of the neural network, but also the video memory used increases so the accuracy depends on how much VRAM you have available. **Width** and **height** must be multiples of 32.

---
#### CPU_classifier
It's the same code used for GPU_classifier but using CPU. The usage is the same.
To increase deep neural network precision you can modify two variable called **DNN_width** and **DNN_height**.

---
# Special thanks to
Special thanks to the guys who developed this beautiful framework and also made it open source -> https://pjreddie.com/darknet/
Thanks also to https://github.com/LucreziaT and https://github.com/dtessaro that have developed this project with me.