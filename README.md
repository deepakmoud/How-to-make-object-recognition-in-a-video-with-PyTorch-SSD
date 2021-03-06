### A regarder et à lire : 

* Object Detection with SSD in the street: https://youtu.be/6q-DBCPROA8
* Deep Learning for Object Detection, A Comprehensive Review by Joyce Xu : https://medium.com/towards-data-science/deep-learning-for-object-detection-a-comprehensive-review-73930816d8d9
* Object Detection-Section 2 Computer Vison A-Z : https://www.udemy.com/computer-vision-a-z/learn/v4/t/lecture/8127602?start=0
* Case Study Self Driving Car : http://bit.ly/2yD1s44 


### A retenir du code : 
```python
# Object Detection

# Importing the libraries
import torch
from torch.autograd import Variable
import cv2
from data import BaseTransform, VOC_CLASSES as labelmap # Data Processing 
from ssd import build_ssd # Import the ssd algoritm 
import imageio # Image processing 

# Defining a function that will do the detections
def detect(frame, net, transform): # inputs : frame of the video, ssd network, transformation of the data 
    height, width = frame.shape[:2]
    frame_t = transform(frame)[0]
    x = torch.from_numpy(frame_t).permute(2, 0, 1)
    x = Variable(x.unsqueeze(0))
    y = net(x) #feed the neural network ssd with the image and we get the output y.
    detections = y.data 
    scale = torch.Tensor([width, height, width, height])
    # detections = [batch, number of classes, number of occurence, (score, x0, Y0, x1, y1)]
    for i in range(detections.size(1)): # i is the class = identified object 
        j = 0 # j is the occurence of the object = number of identified object 
        while detections[0, i, j, 0] > = 0.6:  # threshold 0.6 if not superior at 0.6 = no detection. 
            pt = (detections[0, i, j, 1:] * scale).numpy()
            cv2.rectangle(frame, (int(pt[0]), int(pt[1])), (int(pt[2]), int(pt[3])), (255, 0, 0), 2) # draw the rectangle
            cv2.putText(frame, labelmap[i - 1], (int(pt[0]), int(pt[1])), # put the label of the object 
            cv2.FONT_HERSHEY_SIMPLEX, 2, (255, 255, 255), 2, cv2.LINE_AA)
            j += 1 # incrementation of while 
    return frame  # output : frame with the rectangle and the label 

# Creating the SSD neural network
net = build_ssd('test') # creation of the network 
net.load_state_dict(torch.load('ssd300_mAP_77.43_v2.pth', map_location = lambda storage, loc: storage)) 
# load the pre-trained model

# Creating the transformation
transform = BaseTransform(net.size, (104/256.0, 117/256.0, 123/256.0))

# Doing some Object Detection on a video
reader = imageio.get_reader('funny_dog.mp4') # load the video (if yout want to make object detection on another vidéo, change this line of code) 
fps = reader.get_meta_data()['fps'] # identify the number of frames 
writer = imageio.get_writer('output.mp4', fps = fps) # put the number of frames to te output video
for i, frame in enumerate(reader): # detection for each frame of the video
    frame = detect(frame, net.eval(), transform) 
    writer.append_data(frame)
    print(i)
writer.close()

