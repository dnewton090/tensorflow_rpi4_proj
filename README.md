## Object Detection on RPi 4B (TF 1.15, OpenCV 3.4)

#### This is an update of an existing tutorial provided by @EdjeElectronics...
'https://github.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi

#### Object detection module using MobileNet-SSD deep neural network, trained on Microsoft-COCO dataset. 
* with correct binaries to support *Tensorflow 1.15 on the latest Raspberry Pi Model 4B* running Raspbian Buster10 on *Linux Armv7l architecture*, and fixes for OpenCV compatability.  

After weeks of inspection it finally boiled down to finding a custom binary developed by @PINTO0039 
-- many thanks for your efforts  successful patching TensorFlow issues #15062,#21574,#21855,#23082,#25120,#25748,#29617,#29704,#30359 and eveloping the wheel for Armv7l.
 
TF2.x currently has issues with requiring a Hadoop implementation, whilst PyPI support is only up to TFv1.14.

#### Code

Firstly, install packages for the PiCam, ensuring the ribbon cable is secure with the black tabs in place, and blue tape facing the ethernet ports
```
sudo apt-get update
sudo apt-get install python-picamera python3-picamera
```

Test the PiCam, ensure camera is enabled inside the configuration file
```
sudo raspi-config
(5) interfacing options
(3) camera enabled [optionally enable SSH and VNC Server as well]
```

Test a still shot with output file test.jpg
```
raspistill -o test.jpg
```
For more information https://www.raspberrypi.org/documentation/raspbian/applications/camera

Download PIP and the Python VirtualEnvironoment
```
sudo apt update
sudo apt-get install python3-pip
sudo pip3 install virtualenv 
```
Name and activate the virtual environment (venv)
```
python3 -m virtualenv venv 
source venv/bin/activate
```
 
Update Linux packages:
```
sudo apt update
sudo apt-get install -y libhdf5-dev libc-ares-dev libeigen3-dev gcc gfortran python-dev libgfortran5 \
                        libatlas3-base libatlas-base-dev libopenblas-dev libopenblas-base libblas-dev \
                        liblapack-dev cython openmpi-bin libopenmpi-dev libatlas-base-dev python3-dev
```
...and install Python dependencies
* Keras is a Tensorflow API intended design centric to be in a human readable format, minimizes the number of user actions required for common use cases, and it provides clear & actionable error messages.
* Keras_preprocessing help you go from raw data on disk to a tf.data.io.Dataset object that can be used to train a model.
* The h5py h5py package is a Pythonic interface to the HDF5 binary data format. It allows you to store huge amounts of numerical data, and easily manipulate that data into NumPy arrays.
* pybind is intended for seamless operation of Python and C++ v11

```
sudo pip3 install keras_applications==1.0.8 --no-deps
sudo pip3 install keras_preprocessing==1.1.0 --no-deps
sudo pip3 install h5py==2.9.0
sudo pip3 install pybind11
pip3 install -U --user six wheel mock
sudo pip3 uninstall tensorflow
```
Utilize this lifesaving TF wheel update found at the repo @PINTO0309
```
wget https://github.com/PINTO0309/Tensorflow-bin/raw/master/tensorflow-1.15.0-cp37-cp37m-linux_armv7l.whl
sudo pip3 install tensorflow-1.15.0-cp37-cp37m-linux_armv7l.whl
```

Test whether TF install worked correctly
```
python3
import tensorflow as tf
tf.__version__  <!---'1.15' --->
exit()
```

Install OpenCV
-- First, the Unix dependencies
```
sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
sudo apt-get install libxvidcore-dev libx264-dev
sudo apt-get install qt4-dev-tools libatlas-base-dev
```
OpenCV-Python v4 isn't working on Pi4. Install 3.4.6.27
```
pip3 install opencv-python==3.4.6.27
python3
import cv2
cv2.__version__                        <!---'3.4.6'--->
```
Compiling and Installing Google's Protobuf Compiler

* Protobufs are a type of binary serialization, which uses a determined schema to encode & decode data, which can then be compiled into many low-level languages (C/C++ in our case) 
* Given the very large & complex data structures moving through a neural network, protobufs are used to speed up parsing & processing. 
* This steph facilitates the conversion of the serialized protocol buffers into tensors for Tensorflow
```
sudo apt-get install protobuf-compiler
protoc -version                        <!---libprotoc 3.6.1--->
```
Move into your chosen Directory
```
mkdir objdet
cd objdet
```
Download the TensorFlow repository
```
git clone --depth 1 https://github.com/tensorflow/models.git
```
Modify PYTHONPATH by entering ~/.bashrc and substituting your directory name in the following manner at the end of the file
```
sudo nano ~/.bashrc
export PYTHONPATH=$PYTHONPATH:/home/pi/<directory_name>/models/research:/home/pi/<directory_name>/models/research/slim
```
In my case:
```
PYTHONPATH=$PYTHONPATH:/home/pi/objdet/models/research:/home/pi/objdet/models/research/slim
```
Instructs the protocompiler to convert all .proto files generated by the object detection model into python output:
```
cd /home/pi/objdet/models/research
protoc object_detection/protos/*.proto --python_out=.
```
Download SSDLite-MobileNet model and unpack it into the object_detection directory under your current folder structure objdet/models/object_detection
```
wget http://download.tensorflow.org/models/object_detection/ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz
tar -xzvf ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz
```
Download the Object_detection_picamera.py file 
into the object_detection directory
* Ensure the changes to deprecated TFv1 statements updated as per this repo
```
wget https://raw.githubusercontent.com/dnewton090/tensorflow_rpi4_proj
wget https://raw.githubusercontent.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi/master/Object_detection_picamera.py
```
Et voilá, try to running the Script **on the Raspbian GUI** if there is a network communication error via SSH
* Worrking on a solution here

```
python3 Object_detection_picamera.py 
```
...and for a UBScam...

```
python3 Object_detection_picamera.py --usbcam
```

Other thanks go threads mentioning by @lheontra for his work developing TF 2.x binaries for Arm -- muito obrigado por seus esforços continuous -- and driving me to inspect how binaries were constructed. 

Detailed instructions to come...

DN
