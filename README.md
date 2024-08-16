# AWS Greengrass Docker for MultiTech

<div>

[![My Skills](https://skillicons.dev/icons?i=aws,docker,linux,windows,apple,github)](https://skillicons.dev)

## Overview

<div>

Utilizing Docker containers to install AWS IoT Greengrass on a MultiTech Conduit gateway.

- Enable AWS Lambda functions to run on the gateway.
    - Local data processing
- Reduce latency and bandwidth costs
    - Cloud connectivity efficiency and throughput simplification
    - Eliminate waiting for cloud analysis
- Offline operation
    - Run deployed applications without internet connectivity
    - Store and forward data to the cloud when connectivity is restored
- Device security and management
    - Secure device communication through encryption
- Scalability
    - Deploy and manage multiple devices at scale
- Automation
    - Automatically shutdown devices if issues are detected

</div>

## Getting started

<div>

### Prerequisites

<div>

- MultiTech Conduit gateway
- Docker installed on the gateway
- AWS & Docker account
- Linux, Windows(WSL2 recommended) or MacOS Computer with access to the gateway

</div>

### Computer Installation

<div>

1. Setup Docker on your PC
    - [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/)
    - [Docker Desktop for MacOS](https://docs.docker.com/docker-for-mac/install/)
    - [Docker for Linux](https://docs.docker.com/engine/install/)
        - Or use the following command:
      ```bash
      sudo apt update
      sudo apt install docker.io
      ```
    - Optional: Install Docker Compose
        - [Docker Compose Installation](https://docs.docker.com/compose/install/)
        - Or use the following command:
          ```bash
          sudo apt install docker-compose
          ```
2. Login to Docker
    ```bash
    sudo docker login
    ```
   You will be prompted to enter your Docker credentials:
    ```bash
    Username: your-docker-hub-username
    Password: your-docker-hub-password
    ```
3. MQTT Client Installation
    - Install an MQTT client such as `mosquitto`
        - [Mosquitto Installation](https://mosquitto.org/download/)
        - Or use the following command:
          ```bash
          sudo apt-get install mosquitto
          ```
          ```bash
          mosquitto_sub -t lora/+/+ -v -h 192.168.2.1
          ```
        - Then reboot Mdot
4. Install AWS Cli
    - [AWS CLI Installation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
    - Or use the following command:
      ```bash
      aws configure
      ```
        - You will be prompted to enter your AWS credentials:
        - AWS Access Key ID: YOUR_ACCESS_KEY_ID
        - AWS Secret Access Key: YOUR_SECRET_ACCESS_KEY
        - Default region name: Your region
        - Default output format (Optional): json
        - You can find this information in the AWS Management Console->User Account->Security Credentials->Access Keys->
          Create access key.
            - [AWS Security Credentials](https://console.aws.amazon.com/iam/home?#/security_credentials)
5. Clone the AWS Git Repository for Greengrass
    - [AWS Greengrass Docker](https://github.com/aws-greengrass/aws-greengrass-docker.git)
        ```bash
        git clone git@github.com:aws-greengrass/aws-greengrass-docker.git
        ```
    - Navigate to the cloned repository
      ```bash
      cd aws-greengrass-docker
      ```
6. Configure the Docker File
    - Open the Dockerfile in a text editor or run the following command:
      ```bash
      nano Dockerfile
      ```
        - Update the Dockerfile with the following:
            ```Dockerfile
          ARG GREENGRASS_RELEASE_VERSION=<specified_version>
          ```
        - Optional:
            - Update the Dockerfile with the following:
                ```Dockerfile
              ENV PROVISION=true
              ENV AWS_REGION=<Specified_region>
              ENV THING_NAME=<Specified_thing_name>
              ENV THING_GROUP_NAME=<Specified_thing_group_name>
              ```
            - If you have additional configurations, you can add them to the `RUN` command in the Dockerfile.
    - Save the Dockerfile and exit the text editor.
7. Build the Docker Image
    ```bash
    sudo docker build -t "platform/aws-iot-greengrass:<specified_version>" ./
    ```
8. Run the Docker Container
    ```bash
    sudo docker run --name greengrass -d platform/aws-iot-greengrass:<specified_version>
    ```
    - If using Docker compose:
    - Create a `docker-compose.yml` file with the following:
      ```yaml
      version: '<specified_version>'
      services:
        greengrass:
          image: platform/aws-iot-greengrass:<specified_version>
          environment:
            - PROVISION=true
            - AWS_REGION=<Specified_region>
            - THING_NAME=<Specified_thing_name>
            - THING_GROUP_NAME=<Specified_thing_group_name>
          volumes:
            - /greengrass/certs:/greengrass/v2
          container_name: greengrass
          restart: always
      ```
    - Run the Docker Compose
        ```bash
        sudo docker-compose up -d
        ```
9. Verify the Docker Container is Running
    - Check if the container is running properly
        ```bash
        sudo docker ps
        ```
    - Check the logs of the container
        ```bash
        sudo docker logs greengrass
        ```
10. Push the Docker Image to Docker Hub
     ```bash
     sudo docker push platform/aws-iot-greengrass:<specified_version>
     ```

</div>

### Gateway Installation

<div>

1. SSH into the gateway
    ```bash
    ssh <username>@<gateway_ip>
    ```
2. Ensure that the gateway has an MQTT client install such as `mosquitto`
    - Install `mosquitto` on the gateway
        ```bash
        sudo apt-get install mosquitto
        ```
3. Configure `mosquitto` to run on the gateway
    - navigate to the /etc/mosquitto directory
        - enter the following command:
            ```bash
            sudo nano mosquitto.conf
            ```
        - add a "#" in from of the following lines to the file:
        - "bind_address 127.0.0.1"
        - save and exit nano
        - restart the mosquitto service with the following command:
          ```bash
          /etc/init.d/mosquitto restart
          ```
        - exit the ssh session.
4. Verify that Docker is installed on the gateway
    ```bash
    sudo docker --version
    ```
5. Check if the Docker Daemon is running
    ```bash
    sudo dockerd
    ```
   If the Docker Daemon is not running, start the Docker Daemon manually
    ```bash
    sudo /usr/bin/dockerd &
    ```
6. Login to Docker
    ```bash
    sudo docker login
    ```
   You will be prompted to enter your Docker credentials:
    ```bash
    Username: your-docker-hub-username
    Password: your-docker-hub-password
    ```
7. If you would like to configure the Docker Container on the Conduit gateway after step 5 from the section above:
    ```bash
    scp -r aws_greengrass-docker/ username@remote_host:/path/to/target/directory
    ```
    - Continue through step 7 above:
    - Run the Docker Container
      ```bash
      sudo docker run --rm --init -it --name aws-iot-greengrass \
      --entrypoint /greengrass-entrypoint.sh \
      -v /greengrass/v2:/greengrass/v2 \
      -p 8883:8883 platform/aws-iot-greengrass:v2.4.0
      ```
8. Otherwise, pull the Docker Image from Docker Hub
    ```bash
    sudo docker pull your-dockerhub-username/aws-iot-greengrass:<specified_version>
    ```
   Run the Docker container
    ```bash
    sudo docker run --name greengrass -d your-dockerhub-username/aws-iot-greengrass:<specified_version>
    ```
9. Verify the Docker Container is Running

- Check if the container is running properly
    ```bash
    sudo docker ps
    ```
- Check the logs of the container
    ```bash
    sudo docker logs greengrass
    ```

</div>

### AWS Setup

<div>

1. If during the Docker setup you set the `PROVISION` to `true`, all of the resources you need will be created
   automatically.
2. If you would like to create the resources manually:
    - Login to the AWS console
        - Navigate to the AWS IoT Core
        - On the left hand side find the manage panel
        - Select All devices->Things->Create Thing->Single Thing
            - Name your thing and assign a thing group if you are managing multiple devices
        - Create a new certificate
            - Download the certificate, public key, and private key
            - Activate the certificate in the AWS console
            - Transfer the keys to the gateway
              ```bash
              scp -r /path/to/certificates username@remote_host:/path/to/target/directory
              ```
        - Create and attach an IoT policy to the certificate
        - Example Policy:
          ```json
           {
             "Version": "2012-10-17",
             "Statement": [
               {
                 "Effect": "Allow",
                 "Action": [
                   "iot:Connect",
                   "iot:Publish",
                   "iot:Subscribe",
                   "iot:Receive"
                 ],
                 "Resource": "*"
               }
             ]
           }
          ```
        - Edit the MQTT client configuration file
            - Open the configuration file in a text editor
              ```bash
              nano /etc/mosquitto/mosquitto.conf
              ```
            - Modify the file:
              ```bash
              mosquitto_pub -h <your-iot-endpoint>.amazonaws.com -p 8883 \
              --cafile AmazonRootCA1.pem \
              --cert your-device-cert.pem.crt \
              --key your-private-key.pem.key \
              -t 'test/topic' -m 'Hello from Conduit 300' \
              --tls-version tlsv1.2
              ```
            - Save and exit the text editor
            - Restart the mosquitto service
              ```bash
              /etc/init.d/mosquitto restart
              ```
        - Link the Thing to Greengrass
            - In the AWS IoT Greengrass console, go to your Greengrass Group and add the Conduit 300 as an IoT Thing.
            - Create Subscriptions:
                - Set up subscriptions in the Greengrass Core that allow data to flow between the Conduit 300 and the
                  Lambda functions or other components running on Greengrass.
                - For example, you can create a subscription for the Conduit 300 to publish sensor data to a specific
                  topic that a Greengrass Lambda function subscribes to.
        - Testing
            - On the Conduit 300, use the MQTT client to publish a test message to the AWS IoT topic configured in
              Greengrass.
            - Verify that the message is received by Greengrass and processed by the Lambda function (or other
              configured components).
            - Use the AWS IoT Core console to monitor the connection status and data being sent by the Conduit 300.
            - Check the logs on both the Conduit 300 and the Greengrass Core to ensure there are no errors and that
              communication is established.
          
</div>
</div>

</div>
