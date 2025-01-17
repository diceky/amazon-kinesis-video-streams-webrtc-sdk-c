# Raspberry Pi Remote Controlled Rover with Amazon Kinesis

A remote controlled rover with live camera stream using Raspberry Pi + Pi camera module.
By using [Amazon Kinesis Video Streams C WebRTC SDK](https://github.com/diceky/amazon-kinesis-video-streams-webrtc-sdk-c), the rover can be viewed and controlled from anywhere with a browser, even from your phone.

## Demo

![Demo](readme/rover-amazon-kinesis.gif)

## What you need

- Raspberry Pi 4, 2GB is recommended for optimal performance. However you can use a Pi 3 or older, you may see a increase in latency.
- Raspberry Pi 4 Camera Module or Pi HQ Camera Module (Newer version)
- Python 3 recommended.
- [RC car kit](https://www.amazon.co.jp/gp/product/B088NMV7C6/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)
- [DC motor drivers](https://www.amazon.co.jp/gp/product/B08B87WWHV/ref=ppx_yo_dt_b_asin_title_o04_s00?ie=UTF8&psc=1)
- A battery box + 8 AAA batteries (12V)

## Library Dependencies

```
sudo apt update
sudo apt install -y cmake \
                    libssl-dev \
                    libcurl4-openssl-dev \
                    liblog4cplus-dev \
                    libgstreamer1.0-dev \
                    libgstreamer-plugins-base1.0-dev \
                    gstreamer1.0-plugins-base-apps \
                    gstreamer1.0-plugins-bad \
                    gstreamer1.0-plugins-good \
                    gstreamer1.0-plugins-ugly \
                    gstreamer1.0-tools
```

## Step 0 - Electronics

This code uses the following Raspberry Pi GPIO pins for the 4 motors + LED.
The pin numbers are BCM.

```
#define EN_A 13
#define EN_B 12
#define IN1 22
#define IN2 27
#define IN3 24
#define IN4 23
#define LED 17
```

## Step 1 - Clone this repo

```
git clone https://github.com/diceky/amazon-kinesis-video-streams-webrtc-sdk-c.git
```

(Make sure to use the default `dice/dCampRover` branch)

## Step 2 - Build

```
cd amazon-kinesis-video-streams-webrtc-sdk-c/
mkdir build
cd build
cmake ..
make
```

## Step 3 - Install + configure AWS CLI

```
sudo pip3 install awscli --upgrade
aws configure
```

AWS Access Key ID: xxxxxxxx  
AWS Secret Access Key: xxxxxxx  
Default region name: ap-northeast-1  
Default output format: json

## Step 4 - Set up AWS Policies/Roles/IoT

Follow instructions on **Step 5** and **Step 6** [here](https://aws.amazon.com/jp/builders-flash/202109/angle-control-camera/?awsf.filter-name=*all).

If you are setting up multiple rovers, you can start from **Step 6-3** for the second rover and onwards.

## Step 5 - Install AWS Iot Root Certificate

```
wget https://www.amazontrust.com/repository/AmazonRootCA1.pem -O ~/amazon-kinesis-video-streams-webrtc-sdk-c/build/AmazonRootCA1.pem
```

## Step 6 - Set permanent environment variables for su

Edit `/etc/environment` and add the following 2 lines then reboot the Pi.

```
export AWS_IOT_CREDENTIALS_ENDPOINT="endpointAddress from Step 4"
export AWS_DEFAULT_REGION="ap-northeast-1"
```

## Step 7 - Run video streaming

```
sudo su -
cd /home/pi/amazon-kinesis-video-streams-webrtc-sdk-c/build
./kvsWebrtcClientMasterGstSample raspi-channel
```

## Step 8 - View streaming + send commands via browser

Make a new User in [AWS IAM console](https://aws.amazon.com/iam/), attach `AmazonKinesisVideoStreamsFullAccess` to it and get the Access key ID and secret key.

Go to `client/app.js` in this repo and replace the first 3 lines with the credentials you just retreived + channel name.

Then open up `client/index.html` in your browser and you will be able to view the video stream + send commands to control the rover.
