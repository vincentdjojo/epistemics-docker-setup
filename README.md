# Summary of Steps to Get the Epistemics Repository Running with GUI on MacBook Pro M1 (2020)

Here's a condensed guide based on the troubleshooting and setup process we went through. This assumes basic Terminal familiarity and focuses on the key prerequisites, configurations, and fixes for Docker, ARM64 compatibility, port conflicts, and GUI forwarding via XQuartz. The goal was to run the Java-based epistemics belief system (with Silicon Coppélia affective computing) using Docker orchestration.

## 1. Install Prerequisites

- **Docker Desktop**: Download the Apple Silicon version from docker.com and install. It handles ARM64 natively but emulates AMD64 images as needed.
- **XQuartz**: Download from xquartz.org (ARM-compatible). After install, open Preferences > Security and check "Allow connections from network clients". Enable TCP listening: Quit XQuartz, run `defaults write org.xquartz.X11 nolisten_tcp 0` in Terminal, then relaunch.
- **Git**: Install via `xcode-select --install` or Homebrew (`brew install git` after installing Homebrew from brew.sh).
- Optional: Homebrew for easier package management.

## 2. Clone Repositories

```bash
git clone https://github.com/vincentdjojo/epistemics-docker-setup/
cd docker/epistemics-docker-setup
```

## 3. Initial Docker Setup and Fixes

- Create local settings: `./bin/create-local-settings.sh`.
- Edit credentials if needed: `nano epistemics/etc/credentials-local.sh` (defaults like username/password "selemca" often work).
- Prepare volume: `./prepare-volume.sh`.
- Handle `docker-compose` deprecation: Add alias in `~/.zshrc`: `alias docker-compose='docker compose'`, then `source ~/.zshrc`.
- Manage Swarm mode (initially optional but caused network errors): Run `docker swarm init` for attachable networks, or `docker swarm leave --force` for standalone; we ended up using standalone.
- Make network attachable: In `docker-compose.yml`, update `networks: robopop:` to include `driver: bridge` and `attachable: true`.

## 4. Edit docker-compose.yml for Compatibility and Single Instance

- Remove Swarm-specific `deploy:` sections (e.g., `replicas: 2` for epistemics to avoid port bind conflicts; default to 1 instance).
- Remap ports if conflicts (e.g., epistemics from 8888 to 8899 via `published: 8899`).
- For GUI: Ensure under `silicon-coppelia:`: `environment: - DISPLAY=host.docker.internal:0` and add volume `- ~/.Xauthority:/root/.Xauthority:ro` for X11 auth.
- Handle ARM64: Images are AMD64; ignore warnings (emulation works) or add `platform: linux/amd64` per service. For MySQL, optionally switch to `mariadb:10.6` for native ARM64.
- Remove obsolete `version: '3.3'` to silence warnings.
- Example simplified YAML snippet for silicon-coppelia (GUI focus):

```yaml
silicon-coppelia:
  image: robopop/silicon-coppelia-gui:latest
  networks:
    - robopop
  ports:
    - target: 8080
      published: 8078
  environment:
    - DISPLAY=host.docker.internal:0
  volumes:
    - ~/.Xauthority:/root/.Xauthority:ro
```

## 5. Set Environment Variables (Before Starting)

Run these in Terminal for config/GUI:

```bash
export DB_USER=selemca
export DB_PASSWORD=selemca
export DB_SCHEMA=beliefsystem
export DOCKER_REPOSITORY=robopop
export EPISTEMICS_IMAGE_VERSION=1.3.0-snapshot
export SERVER_PORT=8899
export EXTRA_VOLUMES=""
export SICO_SUFFIX=-gui
export SICO_IMAGE_VERSION=latest
export VOLUMES=""
export SICO_SERVER_PORT=8078
export DISPLAY="$(./osx-host.sh):0"
```

- Add to `~/.zshrc` for permanence.

## 6. Start and Troubleshoot

- Start: `docker compose up -d` (detached mode).
- Check status: `docker ps` (all Up, no Exited).
- Logs: `docker compose logs <service>` (e.g., silicon-coppelia for GUI errors).
- Port conflicts: Use `lsof -i :<port>` (e.g., :8899) to find/kill processes (`kill <PID>`).
- GUI fixes: Run `xhost +` on host (after `export DISPLAY=:0` if needed). For X11 errors, enable TCP in XQuartz and mount .Xauthority.
- Restart specific service: `docker compose restart silicon-coppelia`.
- Stop: `docker compose down`.

## 7. Access the Running System

- Belief System Admin (Epistemics): http://localhost:8899
- Silicon Coppélia Web: http://localhost:8078
- GUI: Ptolemy/Vergil interface appears in XQuartz (wait 30-60s; restart if no window).

This process involved iterative fixes for Docker compatibility, ports, Swarm, and X11 forwarding due to the repo's age (~2022) and M1 architecture. Total time: Several iterations over errors, but now it's operational for experimenting with the belief system! If anything breaks again, check logs first.

______________________________________________________________________________________________________________________________________

# (OLD INSTRUCTIONS) Docker orchestration of ROBOpop components

Scripts for running ROBOpop components in docker containers using docker-compose / docker swarm mode.

## Preparation
1. MacOS: install Docker for Mac
2. MacOS: install XQuartz (necessary for Ptolemy/silicon coppelia UI in Docker containers running on MacOS)
See [Wiki](https://github.com/robopop/docker/wiki/Ptolemy-GUI) for explanation


## Setup (recommended steps)
1. Clone this repository locally
2. If you haven't used docker compose before, run `docker swarm init`
3. Run `bin/create-local-settings.sh`
4. Edit credentials in `epistemics/etc/credentials-local.sh`
5. Run `docker/prepare-volume.sh` to put default configuration files for the Epistemics component in a Docker volume.

## Running
1. Change to directory `robopop`
2. Run `./docker-compose-up.sh`


### Alternative: Running with Ptolemy GUI
* If you use MacOS: run `./docker-compose-up.sh -v --gui -d "$(./osx-host.sh):0"`
* If you use Linux: run `./docker-compose-up.sh -v --gui`


## Accessing
If you run docker on a Linux system or as Docker for Mac, then the software will be available at:

* [Belief System Admin](http://localhost:8888/beliefsystem-webadmin/)
* [Mental World Admin](http://localhost:8888/mentalworld-webadmin/)
* [Mental World Web Application](http://localhost:8888/mentalworld-webapp/)
* [Silicon Coppélia](http://localhost:8084/)

As well as a REST service for the belief system:

* [Belief System REST service](http://localhost:8888/beliefsystem-rest/)

If you run docker in a virtual machine, you may need to replace `localhost` by the name or IP address of the virtual machine.

The belief system can be exported as a ZIP file using the Belief System Admin web application.
[This sample file](https://github.com/robopop/epistemics/raw/master/Installation/BeliefSystem.zip)
can be imported using the same web application.

The primary way to experiment with these software components is with the Ptolemy framework.
See the [Project Wiki](https://github.com/robopop/docker/wiki) for more information.



