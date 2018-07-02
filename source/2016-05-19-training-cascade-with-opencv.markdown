---
layout: post
title: "Training Cascade with OpenCV"
date: 2016-05-19 15:45:42 +0800
comments: true
tags: [MachineLearning, OpenCV]
---


Platform: Linux/Ubuntu

Preparation:

You may have to prepare two different types of images for training a `Binary Classifier Model`, the positive samples and the negative samples.

Here, we gona to use image database from **UIUC Image Database for Car Detection** to demonstrate how to use OpenCV to detection cars in a image.

<!-- more -->

User should put all positive samples which have the same size into a directory.

    ls ./pos > ./pos_list.info
    ls ./neg > ./neg_list.info

Open `pos_list.info` and you will see the path of images have been written into the info file.

![images](/images/img_for_2016_05_19/files.png)

But it isn't enough. To train the cascade with OpenCV, you should supply with the information where is the object in the image. In this demo, what we want to detect is a car. The information that OpenCV need like this: `image_path num x y w h`, which should be append at the end of the path of a image.

`x y w h` describe a rectangle which identify where is the object that we want to find. `num` describe how many objects in the rectangle.

So, I write a script in Python and this script will help to finish that job.

``` python

[trans_pos_location.py]
fileObj = open("./pos_list.info")
newFile = open("./pos_list_new.info", "a+")

for line in fileObj:
    newFile.write("./pos/" + line[:-1] + " 1 0 0 100 40\n")

fileObj.close()
newFile.close()

[trans_neg_location.py]
import os

fileObj = open("./neg_list.info")
newFile = open("./neg_list_new.info", "a+")

for line in fileObj:
    newFile.write(os.getcwd() + "/neg/" + line)

fileObj.close()
newFile.close()

```

Run the following comand:

    opencv_createsamples -info pos_list_new.info -num 550 -w 48 -h 24 -vec cars.vec

    opencv_traincascade -data data -vec abc.vec -bg neg_list_new.info -numPos 550 -numNeg 500 -numStages 2 -w 48 -h 24


`opencv_createsamples` and `opencv_traincascade` are two tool program with OpenCV. The original positive samples for training are images with 100x40 pixels. For the training process, it will cost a lot of memory, so we resize it into smaller one. With that command, `-w 48 -h 24`, positive images are resized into smaller images which's width is 48 pixels and the height of that is 24 pixels.

Here, we can use this script to test the model that we get.

``` python

import cv2
import numpy
from matplotlib import image

car_cascade = cv2.CascadeClassifier("/home/jasonleaster/Desktop/CarData/TrainImages/data/cascade.xml")

gray = image.imread("/home/jasonleaster/Desktop/CarData/TestImages_Scale/test-1.pgm")


faces = car_cascade.detectMultiScale(gray,
                                    scaleFactor = 1.3,
                                    minNeighbors=5,
                                    minSize=(24, 48),
                                    flags = cv2.cv.CV_HAAR_SCALE_IMAGE)


img = gray

for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x+w, y+h), (0, 255, 0), 2)


from matplotlib import pyplot
import pylab
pyplot.imshow(img, cmap = "gray")
pylab.show()    

```

Result:

![images](/images/img_for_2016_05_19/detected_car1.png)


Reference:
1. http://blog.csdn.net/wuxiaoyao12/article/details/39227189
2. www.youtube.com/watch?v=WEzm7L5zoZE
