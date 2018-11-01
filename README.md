# Raster-to-Vector: Revisiting Floorplan Transformation
By Chen Liu, Jiajun Wu, Pushmeet Kohli, and Yasutaka Furukawa

## Introduction

This paper addresses the problem of converting a rasterized
floorplan image into a vector-graphics representation.
Our algorithm significantly outperforms
existing methods and achieves around 90% precision and
recall, getting to the range of production-ready performance. 
To learn more, please refer to our ICCV 2017 [paper](http://art-programmer.github.io/floorplan-transformation/paper.pdf) or visit our [project website](http://art-programmer.github.io/floorplan-transformation.html).

This code implements the algorithm described in our paper in Torch7.

## Updates
We have a follow-up project which reconstructs floorplans from 3D scans. You can find it [here](https://github.com/art-programmer/FloorNet).

For annotator codes, please see [here](#annotator).
## Requirements

- Please install the latest Torch.
- Please install Python 2.7.
- We used a Nvidia Titan GPU with CUDA 8.0 installed.

### Torch packages
- [nn](https://github.com/torch/nn)
- [cunn](https://github.com/torch/cunn)
- [cudnn](https://github.com/soumith/cudnn.torch)
- [image](https://github.com/torch/image)
- [ffi](http://luajit.org/ext_ffi.html)
- [csvigo](https://github.com/clementfarabet/lua---csv)
- [penlight](https://github.com/stevedonovan/Penlight)
- [opencv](https://github.com/marcoscoffier/lua---opencv) (Probably need to be compiled from source)
- [lunatic-python](https://labix.org/lunatic-python)

### Python packages
- [numpy](http://www.scipy.org/scipylib/download.html)
- [Gurobi](http://www.gurobi.com)
- [OpenCV](https://opencv.org/) (v3.4.1)

## Trained models
To use our trained model, please first download it from [Google Drive](https://drive.google.com/file/d/0B2rs82y7tjKrQk0yRFB3RHVDUXM/view?usp=sharing), and put it under folder "checkpoint/" (or specify the its path via option -loadModel="path to the downloaded model").

Our model is fine-tuned based on the pose estimation network introduced in the paper, "Human pose estimation via Convolutional Part Heatmap Regression". You can downloaded their model [here](https://www.adrianbulat.com/human-pose-estimation) (the MPII one), and put it under folder "PoseEstimation/" (or specify the its path via option -loadPoseEstimationModel)

## Data
Our vector-graphics annotation is under "data/" folder. Lists of (raster floorplan image path, vector-graphics annotation path) pairs can be found in either "train.txt", "val.txt", or "test.txt'.

Each row in vector graphics annotations contains (x_min, y_min, x_max, y_max, category, dump_1, dump_2). Category can be either a wall, a door (opening in the paper), a specific icon type, or a specific room type. For walls and doors, two points, (x_min, y_min) and (x_max, y_max), form a line. For icons, x_min, y_min, x_max, and y_max specify a rectangle. For rooms, however, x_min, y_min, x_max, and y_max are unfortunately not for the bounding box of the room, as a room can be of arbitrary shape instead of a rectangle. So, x_min, y_min, x_max, and y_max just denote an arbitrary region which falls inside the room. Please refer to the data loader code to see how to process such annotations.

Here is the [link](https://drive.google.com/file/d/1Ltn5kzzwhvXz6EStI98Zagfq-mI2pf0m/view?usp=sharing) to 100,000+ vector-graphics representation generated by our algorithm. You might want to get 3D popup models from the text files using the popup code below or draw 2D rendering images using the function *fp_ut.drawRepresentationImage(floorplan, representation)* (please see predict.lua for an example).

## Annotator
The code for the annotator is available under folder *annotator/*. You can find a similar annotator written using Python [here](https://github.com/art-programmer/FloorplanAnnotator).

## Usage
To train the network from the pretrained pose estimation network, simply run
```bash
th main.lua -loadPoseEstimationModel  "path to the downloaded pose estimation model"
```

To load our trained model and resume training, please run
```bash
th main.lua -loadModel  "path to the downloaded pretrained model"
```

Here are som useful options for the main script:
- -batchSize  specifies the batch size
- -LR specifies the learning rate
- -nEpochs  specifies the number of epochs
- -checkpointEpochInterval  specifies the number of training epochs between two checkpoints (useful if you want to save less number of checkpoints instead of saving one checkpoint for every epoch)
- useCheckpoint specifies how the training resumes
  * -1: starting from the beginning even when checkpoints previously trained are found
  * 0 (default) resuming from checkpoints if found
  * n (n > 0) resuming from the nth checkpoint

To make prediction on a floorplan image, run
```bash
th predict.lua -loadModel "model path" -floorplanFilename "path to the floorplan image" -outputFilename "output filename"
```
Note that the above script will produce the vectorization result (saved in ".txt" file), the rendering image (saved in ".png" file), and a text file which could be used for generating 3D models (saved in "_popup.txt").

To evaluate performance on the benchmark, run
```bash
th evaluate.lua -loadModel "model path" -resultPath "path to save results"
```

## Generate 3D models
Automatic 3D model generation based on our vectorization results is implemented in both C++ (under folder popup/) and Python (under folder rendering/).

For the C++ code, run the following:
```bash
cd popup/code/
cmake .
make
./popup_cli ../data/floorplan_1.txt
```

The data file (e.g., popup/data/floorplan_1.txt), which could be generated by *predict.lua* (\*_popup.txt), has the following format:

```csv
width height
the number of walls
(Wall descriptions)
x_1, y_1, x_2, y_2, room type on the left, room type on the right
...
(Opening descriptions)
x_1, y_1, x_2, y_2, 'door', dummy, dummy
(Icon descriptions)
x_1, y_1, x_2, y_2, icon type, dummy, dummy
```

You could optionally use the corresponding input raster image or the final vector-graphics rendering as the texture image for the floor. To do so, please put the image under the data folder and rename it to the same name with the data file with suffix ".png" (e.g., floorplan_1.png).

The Python code is based on Panda3D. First enter folder rendering/, and then either run:

```bash
python viewer.py
```

to view a 3D model, or run:

```bash
python rendering.py
```
to render one view of the 3D model given camera pose. Please check the code to see how to specify the model to view and how to render different views.

## Contact

If you have any questions, please contact me at chenliu@wustl.edu.
