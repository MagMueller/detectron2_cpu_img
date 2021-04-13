# detectron2_cpu_img
This is 'development' docker container:
* detectron2 for CPU
* python3.7 and libs
* torch 1.8.1
* simple web service, which get image as input and returns detectron2 instanseSegmentation result and exif info (Python/Flask)

# Note
I don't find 'ready to use' solution for my task (detect objects on image) so I had to build it

I don't know much about CV/image recognition/python/ docker, so there may be errors and duck code. __Any help is welcome__.

I don't have CUDA GPU, so I build CPU based service which is more slower.

# Usage
Build and run docker image:

`docker-compose up -d` or `docker-compose up -d --build` (to re-build image)

Service (`<your_host>:<your_port>/api/v1.0/imgrecognize/`) will start with docker up. Some time will spend for segmentation model downloading at start up (once per model since last image build)

Open in browser  `http://<your_host>:<your_port>/`
Select image and push it. Get result.

      or

Post image any way you preffer, some thing like:

`curl --request POST -F "file=@IMG.JPG" localhost:5000/api/v1.0/imgrecognize/`

-- get json as result

*You can post more than one image in request, but procesing may take too much time and connection will close with time-out*

Add request params to URL if needed:
* _exif_ - to return exif if exist (False is default, any other value is eq True)
* _resimg_ - to return result image with objects marked as base64 string (False is default, any other value is eq True)
* _autorotation_ - autorotate image using exif data (Orientation) (False is default, any other value is eq True)
* _rotation=<value>_ - rotate to <value> degrees before analisys. Works with/without _autorotation_
* _resize=<value>_ - resize to <value> px (max side of image) before analisys. If value not passed: 1000px is default.

`curl --request POST -F "file=@IMG.JPG" localhost:5000/api/v1.0/imgrecognize/?exif=False&autorotation&rotation=90`

## Docker-compose
Some variables can be passed throw docker-compose.yml file
```
      - SEGMENTATION_MODEL=COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml
      - FLASK_DEBUG=True
      - FLASK_HOST=0.0.0.0
      - FLASK_PORT=5000
```
Find more segmentation models at `https://github.com/facebookresearch/detectron2/tree/master/configs/COCO-InstanceSegmentation`
# Performance
My HW instanse is
* Xeon E3 1260L
* DDR3 16Gb
* HDD 3Tb WD Red

Software:
* VM (Proxmox) with 6 cores (host) and 1.5Gb Ram
* Ubuntu 20.04 inside VM
* Docker 20.10.5

Results highly depends on image resolution: less resolution - faster analisys (but recognition still good) 
```
 time  `curl --request POST -F "file=@IMG_3448.JPG" 192.168.1.111:5000/api/v1.0/imgrecognize/ > /dev/null`
 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 3452k  100 52765  100 3401k   4940   318k  0:00:10  0:00:10 --:--:-- 15201

real    0m10.698s
user    0m0.009s
sys     0m0.023s
```

# Near future plans:
* clean-up python code
*  _DONE:_ add some variables for request (like `?noexif` or `?noobjects`)
* add basic-auth for service
* _DONE_: add posibility use not only segmentation models
* _DONE_: add possibility to resize image (__panoptic segmentation causes docker crash if input image too big__, may be to low RAM, so resize image is good point to solve this and increase recognition speed
