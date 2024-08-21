# Ansible Playbook for Nginx and FFmpeg

## Overview

This Ansible playbook sets up an Nginx web server and an FFmpeg stream processing server on a Linux host using Docker. The Nginx server serves the latest frame from a live stream as the root index.

## Project Structure

The project structure is as follows:

```
ansible-pet/
├── ansible.cfg
├── inventory
│   ├── default
│   │   ├── configs
│   │   │   └── vars.yml (Optional)
│   │   ├── group_vars
│   │   │   └── all.yml
│   │   └── host.ini
│   └── production
│       ├── configs
│       │   └── vars.yml (Optional)
│       ├── group_vars
│       │   └── all.yml
│       └── host.ini
├── README.md
├── requirements.txt
├── roles
│   ├── ffmpeg
│   │   └── tasks
│   │       └── main.yml
│   └── nginx
│       ├── tasks
│       │   └── main.yml
│       └── templates
│           └── default.conf.j2
├── server_down.yml
└── site.yml
```

- `ansible.cfg`: Configuration file for Ansible, specifying default behavior such as inventory location and privilege escalation.
- `inventory/`: Directory containing inventory files and configuration for different environments.
  - `default/`: The default environment setup.
    - `group_vars/all.yml`: Variables applied to all hosts in the default environment.
    - `host.ini`: The inventory file listing the hosts for the default environment.
  - `production/`: The production environment setup.
    - `group_vars/all.yml`: Variables applied to all hosts in the production environment.
    - `host.ini`: The inventory file listing the hosts for the production environment.
- `roles/`: Directory containing the roles for Nginx and FFmpeg.
  - `roles/nginx/tasks/main.yml`: Tasks for setting up and running the Nginx container.
  - `roles/nginx/templates/default.conf.j2`: Nginx configuration template.
  - `roles/ffmpeg/tasks/main.yml`: Tasks for setting up and running the FFmpeg container.
- `server_down.yml`: Playbook to tear down the setup and remove all configurations and containers.
- `site.yml`: The main playbook that includes all roles and applies the configurations.

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

### Step 2: Set Up the Python Environment
```sh
python3 -m venv env
source env/bin/activate
pip install -r requirements.txt
```

### Step 3: Configure Your Inventory

You can set up different environments by creating custom inventory directories inside the `inventory/` folder. Each environment should have its own `group_vars/all.yml` file for environment-specific variables and a corresponding `host.ini` file.

Example for setting up a production environment:

```
inventory/
├── production
│   ├── group_vars
│   │   └── all.yml
│   └── host.ini
```

- **`host.ini`**: Contains the IP addresses or hostnames of the servers for the environment. Example:

```ini
[webservers]
prod_server_ip ansible_user=your_user
```

- **`group_vars/all.yml`**: Environment-specific variables that apply to all hosts in this environment.

Example `all.yml`:

```yaml
# Nginx configuration variables
nginx_container_name: nginx-ffmpeg-prod
nginx_image: nginx:stable-alpine
nginx_config_path: /home/{{ ansible_user }}/nginx-ffmpeg-prod/default.conf
nginx_html_path: /home/{{ ansible_user }}/nginx-ffmpeg-prod/html

# FFmpeg configuration variables
ffmpeg_container_name: ffmpeg-prod
ffmpeg_image: jrottenberg/ffmpeg:latest
ffmpeg_command: >
  -i http://webcam.mchcares.com/mjpg/video.mjpg -vf fps=1 /frames/index.jpg -y

# Port configuration
nginx_ports:
  - "80:80"
```

### Step 4: Update Your SSH Access

Make sure your public SSH key is copied to the target hosts:

```sh
ssh-copy-id {{your_user}}@{{your_server_ip}}
```

### Step 5: Run the Playbook

Run the Ansible playbook to set up Nginx and FFmpeg on your target host. Specify the inventory file for your environment:

For the default environment:

```sh
ansible-playbook -i inventory/default/host.ini site.yml 
```

For a custom environment like production:

```sh
ansible-playbook -i inventory/production/host.ini site.yml 
```

### Step 6: Access the Live Stream

Open a web browser and navigate to the public IP address of your target host:

```
http://your_server_ip/
```

The page will display the latest frame from the live stream.

### Step 7: Tearing Down the Setup

To remove all the running instances and configurations from the target host, run the `server_down.yml` playbook:

```sh
ansible-playbook -i inventory/production/host.ini server_down.yml
```

## Configuration Options 

### FFmpeg & Nginx Container Setup Guide

This guide provides a detailed explanation of the variables used for setting up FFmpeg and Nginx containers in an Ansible playbook. These variables are defined in the `group_vars/all.yml` file for each environment and are used to customize the deployment.

#### Variables Overview

| Variable Name             | Description                                                                                 | Example                                              |
|---------------------------|---------------------------------------------------------------------------------------------|------------------------------------------------------|
| **`nginx_container_name`** | The name of the Nginx container.                                                            | `nginx-ffmpeg-prod`                                  |
| **`nginx_image`**          | The Docker image to use for the Nginx container.                                            | `nginx:stable-alpine`                                |
| **`nginx_config_path`**    | The path to the Nginx configuration file on the host machine.                               | `/home/{{ ansible_user }}/nginx-ffmpeg/default.conf`  |
| **`nginx_html_path`**      | The path to the HTML directory on the host machine that will be served by Nginx.            | `/home/{{ ansible_user }}/nginx-ffmpeg/html`         |

### FFmpeg Configuration Variables

| Variable Name               | Description                                                                             | Example                                                |
|-----------------------------|-----------------------------------------------------------------------------------------|--------------------------------------------------------|
| **`nginx_ffmpeg_base_path`** | The base directory for the Nginx and FFmpeg setup on the host machine.                  | `/home/{{ ansible_user }}/nginx-ffmpeg`                |
| **`nginx_ffmpeg_html_path`** | The specific path where FFmpeg will store captured frames, which will be served by Nginx.| `/home/{{ ansible_user }}/nginx-ffmpeg/html`           |
| **`ffmpeg_container_name`**  | The name of the FFmpeg container.                                                       | `ffmpeg-prod`                                          |
| **`ffmpeg_image`**           | The Docker image to use for the FFmpeg container.                                       | `jrottenberg/ffmpeg:latest`                            |
| **`ffmpeg_command`**         | The FFmpeg command to be executed by the container to capture frames from a video stream.| `-i http://webcam.mchcares.com/mjpg/video.mjpg -vf fps=1 /frames/index.jpg -y` |

### Nginx Port Configuration

| Variable Name      | Description                                                          | Example  |
|--------------------|----------------------------------------------------------------------|----------|
| **`nginx_ports`**  | Defines the port mapping between the host and the Nginx container.    | `["80:80"]` |

### How to Use These Variables

1. **Create the Variables File**: 
   - Copy the `group_vars/all.yml` file from `inventory/default/group_vars/` and modify it according to your environment-specific needs.

2. **Update Inventory File**:
   - Ensure your `host.ini` is correctly set up to point to your target hosts for each environment.

3. **Run Playbook with the Specified Inventory**:
   - Use the appropriate inventory path when running your playbooks to target the correct environment.

## Removing Running Instances

To reset your target server to its default state and remove all configurations and containers, run the following command:

```sh
ansible-playbook -i inventory/production/host.ini server_down.yml
```

This will stop and remove all Docker containers and delete related directories and files as configured in your playbook.