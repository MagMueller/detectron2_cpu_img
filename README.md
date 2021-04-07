# detectron2_cpu_img
This is 'development' docker image:
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

Post image any way you preffer, some thing like:

`curl --request POST -F "file=@IMG.JPG" localhost:5000/api/v1.0/imgrecognize/`

-- get json as result

*You can post more than one image in request, but procesing may take too much time and connection will close with time-out*

## Docker-compose
Some variables can be passed throw docker-compose.yml file
```
      - SEGMENTATION_MODEL=COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml
      - FLASK_DEBUG=True
      - FLASK_HOST=0.0.0.0
      - FLASK_HPORT=5000
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
* add some variables for request (like `?noexif` or `?noobjects`)
* add basic-auth for service
* add posibility use not only segmentation models
