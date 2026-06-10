# realtime-object-detection-yolo
**Using realsense d455，based on yolo(v5), for realtime-object-detection and positioning in cmeara frame**

# 1. Environment：
0.(Optional) miniconda for python env
```bash
conda create -n yolo5py38 python=3.8
conda activate yolo5py38
```

1. Requirements for YOLOv5 Python environment. The requirements.txt in yolov5 also works.

```bash
pip install -r requirements.txt
```

2. The RealSense camera and pyrealsense2 libraries

```bash
pip install pyrealsense2
```

3. Run for detection
```
python main_yolo_ros.py
```

**Tested platform**

- **win10** python 3.8 Pytorch 1.10.2+gpu CUDA 11.3  NVIDIA GeForce MX150

- **ubuntu16.04**  python 3.6 Pytorch 1.7.1+cpu

- **ubuntu20.04**  python 3.8 Pytorch 1.8.1+cpu

# 2.Results：
<!--
- Colorimage:

 ![image-20220213144406079](https://github.com/L53317/realtime-object-detection-yolov5-d435i/blob/main/images/image-20220213144406079.png)
 -->

- Colorimage and depthimage example:

<img src="./images/image-20220213143921695.png" height="300">

A demonstration video can be found [here](https://youtu.be/R8SZTFAUvmo).

# 3.Model config：

Modify the configuration of model，according to yolov5s or your own trained model weight and camera.

```yaml
weight:  "weights/yolov5s.pt"
# Image size
input_size: 640
# object categrory
class_num:  80
# label name
class_name: [ 'person', 'bicycle', 'car', 'motorcycle', 'airplane', 'bus', 'train', 'truck', 'boat', 'traffic light',
         'fire hydrant', 'stop sign', 'parking meter', 'bench', 'bird', 'cat', 'dog', 'horse', 'sheep', 'cow',
         'elephant', 'bear', 'zebra', 'giraffe', 'backpack', 'umbrella', 'handbag', 'tie', 'suitcase', 'frisbee',
         'skis', 'snowboard', 'sports ball', 'kite', 'baseball bat', 'baseball glove', 'skateboard', 'surfboard',
         'tennis racket', 'bottle', 'wine glass', 'cup', 'fork', 'knife', 'spoon', 'bowl', 'banana', 'apple',
         'sandwich', 'orange', 'broccoli', 'carrot', 'hot dog', 'pizza', 'donut', 'cake', 'chair', 'couch',
         'potted plant', 'bed', 'dining table', 'toilet', 'tv', 'laptop', 'mouse', 'remote', 'keyboard', 'cell phone',
         'microwave', 'oven', 'toaster', 'sink', 'refrigerator', 'book', 'clock', 'vase', 'scissors', 'teddy bear',
         'hair drier', 'toothbrush' ]

threshold:
  iou: 0.45
  confidence: 0.6
# Selected device
# - cpu
# - 0 <- GPU
device: 'cpu'
```

# 4. Camera config：
It seems that the resolution can only be changed by modifying specific parameters, otherwise an error will occur. For d435i: 1280x720, 640x480, 848x480.
```python
config.enable_stream(rs.stream.depth, 1280, 720, rs.format.z16, 30)
config.enable_stream(rs.stream.color, 1280, 720, rs.format.bgr8, 30)
```
# 5. Code return xyz：
The code below implements the transformation from pixel coordinates to camera coordinates and marks the center point and 3D coordinate information.

```python
for i in range(len(xyxy_list)):
    ux = int((xyxy_list[i][0]+xyxy_list[i][2])/2)  # Pixel x
    uy = int((xyxy_list[i][1]+xyxy_list[i][3])/2)  # Pixel y
    dis = aligned_depth_frame.get_distance(ux, uy)
    camera_xyz = rs.rs2_deproject_pixel_to_point(
    depth_intrin, (ux, uy), dis)  # camera xyz
    camera_xyz = np.round(np.array(camera_xyz), 3)
    camera_xyz = camera_xyz.tolist()
    cv2.circle(canvas, (ux,uy), 4, (255, 255, 255), 5) # mark object center
    cv2.putText(canvas, str(camera_xyz), (ux+20, uy+10), 0, 1,
                                [225, 255, 255], thickness=2, lineType=cv2.LINE_AA) # mark position in camera frame
    camera_xyz_list.append(camera_xyz)
    #print(camera_xyz_list)
```

# 6.TODO
- [ ] Replace with the latest YOLO models, like yolo10 or yolo26；
- [ ] Use for ROS and calculate the position in the robot frame;
- [ ] Small object detection can be improved, detection for the mouse only in 0.5m


# 7.Reference:

[https://github.com/ultralytics/yolov5](https://github.com/ultralytics/yolov5)

[https://github.com/mushroom-x/yolov5-simple](https://github.com/mushroom-x/yolov5-simple)

[https://github.com/Thinkin99/yolov5_d435i_detection](https://github.com/Thinkin99/yolov5_d435i_detection)
