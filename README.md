# Ansible Playbook for Nginx and FFmpeg

## Overview

This Ansible playbook sets up an Nginx web server and an FFmpeg stream processing server on a Linux host using Docker. The Nginx server serves the latest frame from a live stream as the root index.

## Structure

The project structure is as follows:

```
ansible-pet/
├── configs
│   └── vars.yml
├── host.ini
├── README.md
├── requirements.txt
├── roles
│   ├── ffmpeg
│   │   └── tasks
│   │       └── main.yml
│   └── nginx
│       ├── tasks
│       │   └── main.yml
│       └── templates
│           └── default.conf.j2
└── site.yml
```

- `inventory.ini`: Contains the inventory of the servers.
- `site.yml`: The main playbook that includes all roles.
- `roles/`: Directory containing the roles for Nginx and FFmpeg.
- `roles/nginx/tasks/main.yml`: Tasks for setting up and running the Nginx container.
- `roles/nginx/templates/default.conf.j2`: Nginx configuration template.
- `roles/ffmpeg/tasks/main.yml`: Tasks for setting up and running the FFmpeg container.
- `configs/vars.yml`: Variables configuration file (See description under Configuration options)

## Prerequisites

1. Ansible installed on your local machine.
2. Docker installed on the target host.
3. A Linux host.
4. SSH access to the target host.

## Setup and Usage

### Step 1: Clone the Repository

```sh
git clone https://github.com/andranikasd/ansible-pet
cd ansible-pet
```

### Step 2: Setup python environment
```sh
python3 -m venv env
source env/bin/activate
pip install -r requirements.txt
```

### Step 3: Update Inventory File

Edit the `inventory.ini` file to include your target host details.

```
[webservers]
your_server_ip ansible_user=your_user
```

For local setup, use `localhost` as the host:

```
[webservers]
localhost ansible_connection=local
```

### Step 4: Copy you public ssh key to target host
```sh
ssh-copy-id {{your_user}}@{{your_server_ip}}
```

### Step 5: Run the Playbook

Run the Ansible playbook to set up Nginx and FFmpeg on your target host.

```sh
ansible-playbook -i host.ini site.yml --ask-become-pass
```

### Step 5: Access the Live Stream

Open a web browser and navigate to the public IP address of your target host:

```
http://your_server_ip/
```

The page will display the latest frame from the live stream.

### NOTE: 
Additionally You can change some configuration e.x image versions for your specific needs in `configs/vars.yaml`

## Configuration options 

## FFmpeg & Nginx Container Setup Guide

This guide provides a detailed explanation of the variables used for setting up FFmpeg and Nginx containers in an Ansible playbook. These variables are defined in `configs/vars.yml` and are used to customize the deployment according to your environment.

### Variables Overview

| Variable Name             | Description                                                                                 | Example                                              |
|---------------------------|---------------------------------------------------------------------------------------------|------------------------------------------------------|
| **`nginx_container_name`** | The name of the Nginx container.                                                            | `nginx-ffmpeg`                                       |
| **`nginx_image`**          | The Docker image to use for the Nginx container.                                            | `nginx:stable-alpine`                                |
| **`nginx_config_path`**    | The path to the Nginx configuration file on the host machine.                               | `/home/{{ ansible_user }}/nginx-ffmpeg/default.conf`  |
| **`nginx_html_path`**      | The path to the HTML directory on the host machine that will be served by Nginx.            | `/home/{{ ansible_user }}/nginx-ffmpeg/html`         |

### FFmpeg Configuration Variables

| Variable Name               | Description                                                                             | Example                                                |
|-----------------------------|-----------------------------------------------------------------------------------------|--------------------------------------------------------|
| **`nginx_ffmpeg_base_path`** | The base directory for the Nginx and FFmpeg setup on the host machine.                  | `/home/{{ ansible_user }}/nginx-ffmpeg`                |
| **`nginx_ffmpeg_html_path`** | The specific path where FFmpeg will store captured frames, which will be served by Nginx.| `/home/{{ ansible_user }}/nginx-ffmpeg/html`           |
| **`ffmpeg_container_name`**  | The name of the FFmpeg container.                                                       | `ffmpeg`                                               |
| **`ffmpeg_image`**           | The Docker image to use for the FFmpeg container.                                       | `jrottenberg/ffmpeg:latest`                            |
| **`ffmpeg_command`**         | The FFmpeg command to be executed by the container to capture frames from a video stream.| `-i http://webcam.mchcares.com/mjpg/video.mjpg -vf fps=1 /frames/index.jpg -y` |

### Nginx Port Configuration

| Variable Name      | Description                                                          | Example  |
|--------------------|----------------------------------------------------------------------|----------|
| **`nginx_ports`**  | Defines the port mapping between the host and the Nginx container.    | `["80:80"]` |

### How to Use These Variables

1. **Create the Variables File**: 
   - Create a `nginx_vars.yml` file in your Ansible project directory.
   
2. **Populate the Variables File**:
   - Copy the variable definitions into the `configs/vars.yml` file and customize them according to your environment.

   Example `configs/vars.yml`:
   ```yaml
   # nginx configuration variables
   nginx_container_name: nginx-ffmpeg
   nginx_image: nginx:stable-alpine
   nginx_config_path: /home/{{ ansible_user }}/nginx-ffmpeg/default.conf
   nginx_html_path: /home/{{ ansible_user }}/nginx-ffmpeg/html
   nginx_ffmpeg_base_path: /home/{{ ansible_user }}/nginx-ffmpeg
   nginx_ffmpeg_html_path: "{{ nginx_ffmpeg_base_path }}/html"
   ffmpeg_container_name: ffmpeg
   ffmpeg_image: jrottenberg/ffmpeg:latest
   ffmpeg_command: >
     -i http://webcam.mchcares.com/mjpg/video.mjpg -vf fps=1 /frames/index.jpg -y
   nginx_ports:
     - "80:80"
    ```
