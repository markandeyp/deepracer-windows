# Deepracer Windows Setup

## System Requirements

- Windows 10/11
- 100 GB Free hard drive space
- Relatively latest Processor
- Minimum 16 GB RAM
- Good to have a GPU

## Install WSL

- Open a command prompt and 
- Run `wsl.exe --install`

### If you are using NVIDIA GPU

To use NVIDIA GPU for deepracer, you need to configure CUDA on WSL. To do this, follow below steps on a `command prompt`.

```bash
sudo apt-key del 7fa2af80

wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.1-1_all.deb

sudo dpkg -i cuda-keyring_1.1-1_all.deb

sudo apt-get update

sudo apt-get -y install cuda-toolkit-12-5
```

Detailed instructions are mentioned [here](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)

## Configure environment

```bash
sudo apt-get install jq awscli python3-boto3 docker-compose

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update && sudo apt-get install -y --no-install-recommends docker-ce docker-ce-cli containerd.io

distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -

curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get -y install docker-buildx-plugin
```

### Restart wsl

To ensure everything is installed and to make them take effect, restart WSL.

- Open a command prompt and
- Run `wsl --terminate <Distroname>`
- Run `wsl -d <Distroname>`

### Continue configuration

```bash
cat /etc/docker/daemon.json | jq 'del(."default-runtime") + {"default-runtime": "nvidia"}' | sudo tee /etc/docker/daemon.json

sudo usermod -a -G docker $(id -un)
```

## Configure Deepracer

- Clone DRfC github  repo using `git clone https://github.com/aws-deepracer-community/deepracer-for-cloud.git`
- Move to the cloned repository using `cd <path>/deepracer-for-cloud`
- Run `./bin/init.sh -a gpu -c local`. If you don't have a **GPU**, it will automatically switch to **CPU** based configuration.

## Configure Environment variables/Services

DRfC comes with scripts to easily manage training and execution. These scripts can be enabled using following command

- Run `source ./bin/activate.sh`
- You should see a message about successful configuration of `s3_minio` service
- Validate `minio` configuration by running `docker ps` which should list a running container named `s3_minio`
- You can connect to running minio service by visiting [http://{IP_ADDR}:9000](http://{IP_ADDR}:9000) on host machine. 
- Minio credential can be obtained by running `cat ~/.aws/credentials` on WSL terminal. **aws_access_key_id** is *username* and **aws_secret_access_key** is *password*.


## Running training

You can now start training as usually would be done for spot trainings.

- Customise **custom_files** and upload using `dr-upload-custom-files` command
- Start training using `dr-start-training` command
- Stop training using `dr-stop-training` command

## Troubleshooting

If you are using GPU and upon starting the training receive a message **Sagemaker not running**, investigate the logs using 

- `docker ps -a`
- `docker logs <container-name>`

If, sagemaker fails to start with error `unknow runtime nvidia`, verify your `/etc/docker/daemon.json` to make sure `nvidia` is a listed runtime. If `daemon.json` is empty or doesn't define `nvidia` runtime, you can configure as below.

- find **nvidia-runtime** path using `which nvidia-container-runtime` command
- update `/etc/docker/daemon.json` with content below

```json
{
    "runtimes":{
        "nvidia":{
            "path":"<nvidia-container-runtime-path>",
            "runtimeArgs":[
                
            ]
        }
    }
}
```