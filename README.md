# Ansible Playbook for Nginx and FFmpeg

## Overview

This Ansible playbook sets up an Nginx web server and an FFmpeg stream processing server on a Linux host using Docker. The Nginx server serves the latest frame from a live stream as the root index.

## Structure

The project structure is as follows:

```
ansible_docker_setup/
├── inventory.ini
├── site.yml
└── roles/
    ├── nginx/
    │   ├── tasks/
    │   │   └── main.yml
    │   └── templates/
    │       └── default.conf.j2
    └── ffmpeg/
        └── tasks/
            └── main.yml
```

- `inventory.ini`: Contains the inventory of the servers.
- `site.yml`: The main playbook that includes all roles.
- `roles/`: Directory containing the roles for Nginx and FFmpeg.
- `roles/nginx/tasks/main.yml`: Tasks for setting up and running the Nginx container.
- `roles/nginx/templates/default.conf.j2`: Nginx configuration template.
- `roles/ffmpeg/tasks/main.yml`: Tasks for setting up and running the FFmpeg container.

## Prerequisites

1. Ansible installed on your local machine.
2. Docker installed on the target host.
3. A Linux host.
4. SSH access to the target host.

## Setup and Usage

### Step 1: Clone the Repository

```sh
git clone https://github.com/elhakoyan/web-stream.git
cd ansible_docker_setup
```

### Step 2: Update Inventory File

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

### Step 3: Run the Playbook

Run the Ansible playbook to set up Nginx and FFmpeg on your target host.

```sh
ansible-playbook -i host.ini site.yml --ask-become-pass
```

### Step 4: Access the Live Stream

Open a web browser and navigate to the public IP address of your target host:

```
http://your_server_ip/
```

The page will display the latest frame from the live stream.

## Roles

### Nginx Role

Sets up and runs the Nginx container.

`roles/nginx/tasks/main.yml`:

```yaml
---
- name: Create application directory
  file:
    path: /opt/nginx-ffmpeg
    state: directory

- name: Create HTML directory
  file:
    path: /opt/nginx-ffmpeg/html
    state: directory

- name: Copy Nginx configuration file
  template:
    src: default.conf.j2
    dest: /opt/nginx-ffmpeg/default.conf

- name: Run Nginx container
  docker_container:
    name: nginx
    image: nginx:latest
    state: started
    restart_policy: always
    ports:
      - "80:80"
    volumes:
      - /opt/nginx-ffmpeg/default.conf:/etc/nginx/conf.d/default.conf
      - /opt/nginx-ffmpeg/html:/usr/share/nginx/html
```

### FFmpeg Role

Sets up and runs the FFmpeg container to capture frames from the live stream.

`roles/ffmpeg/tasks/main.yml`:

```yaml
---
- name: Create application directory
  file:
    path: /opt/nginx-ffmpeg
    state: directory

- name: Create HTML directory
  file:
    path: /opt/nginx-ffmpeg/html
    state: directory

- name: Run FFmpeg container
  docker_container:
    name: ffmpeg
    image: jrottenberg/ffmpeg:latest
    state: started
    restart_policy: always
    command: >
      -i http://webcam.mchcares.com/mjpg/video.mjpg -vf fps=1 /frames/index.jpg -y
    volumes:
      - /opt/nginx-ffmpeg/html:/frames
```
