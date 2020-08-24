## Object Detection on RPi 4B (TF 1.15, OpenCV 3.4)

#### This is an update of an existing tutorial provided by @EdjeElectronics...
'https://github.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi

...although with correct binaries for Tensorflow 1.15 and fixes for OpenCV compatability.  
After weeks of inspection it finally boiled down to finding a custom binary developed by @PINTO0039 
-- many thanks for your efforts  successful patching TensorFlow issues #15062,#21574,#21855,#23082,#25120,#25748,#29617,#29704,#30359 and eveloping the wheel for Armv7l.
 
TF2.x currently has issues with requiring a Hadoop implementation, whilst PyPI support is only up to TFv1.14.

#### Code
Download PIP and the virtual environment
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
 
Update Linux packages, install python dependencies
```
sudo apt update
sudo apt-get install -y libhdf5-dev libc-ares-dev libeigen3-dev gcc gfortran python-dev libgfortran5 \
                        libatlas3-base libatlas-base-dev libopenblas-dev libopenblas-base libblas-dev \
                        liblapack-dev cython openmpi-bin libopenmpi-dev libatlas-base-dev python3-dev

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
cv2.__version__  <!---'3.4.6'--->
```
Compiling and Installing Google's Protobuf Compiler
•	Protobufs are a type of binary serialization, which uses a determined schema to encode & decode data, which can then be compiled into many low-level languages (C/C++ in our case) 
•	Given the very large & complex data structures moving through a neural network, protobufs are used to speed up parsing & processing. 
• This steph facilitates the conversion of the serialized protocol buffers into tensors for Tensorflow
```
sudo apt-get install protobuf-compiler
protoc -version <!---libprotoc 3.6.1--->
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
I'm still not sure how this ProtoCompiler works, but again substituting your directory into the following:
```
cd /home/pi/objdet/models/research
protoc object_detection/protos/*.proto --python_out=.
```
Download SSDLite-MobileNet model and unpack it into the object_detection directory under your current folder structure objdet/models/object_detection
```
wget http://download.tensorflow.org/models/object_detection/ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz
tar -xzvf ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz
```
Download the Object_detection_picamera.py file into the object_detection directory
```
wget https://raw.githubusercontent.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi/master/Object_detection_picamera.py
```
Et voilá, try to run the Script **on the Raspbian GUI, not via SSH**
```
python3 Object_detection_picamera.py 
```
...and for a UBScam...

```
python3 Object_detection_picamera.py 
```

Other thanks go threads mentioning by @lheontra for his work developing TF 2.x binaries for Arm -- muito obrigado por seus esforcos continuous -- and driving me to inspect how binaries were constructed. 

Detailed instructions to come...

DN
