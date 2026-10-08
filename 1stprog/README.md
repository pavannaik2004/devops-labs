# Program 1 – Build a Docker Container from a Custom Dockerfile

This program packages a small Python **Flask** web app into a Docker image and runs it as a container. The app returns `Hello, Docker!` at `http://localhost:5000`.

## Project Structure

```
1stprog/
├── Dockerfile         # Instructions to build the image
├── requirements.txt   # Python dependencies (flask)
├── app.py             # Flask application
└── README.md
```

## Prerequisites

| Requirement | Notes |
|---|---|
| **Docker** | Docker Desktop (Windows/macOS) or Docker Engine (Linux). On Windows, Docker Desktop needs WSL 2 enabled. |
| **Internet connection** | Needed the first time, to pull the `python:3.9-slim` base image and install Flask. |
| **Free port 5000** | Nothing else should be using port 5000 on your machine. |
| A terminal | PowerShell, Command Prompt, Git Bash or a Linux/macOS terminal. |

You do **not** need Python or Flask installed on your machine; they are installed inside the container.

Check that Docker is installed and running:

```
docker --version
docker info
```

If `docker info` shows an error, start Docker Desktop and wait until it reports that Docker is running.

## How to Execute

Open a terminal **inside the `1stprog` folder** (the folder that contains the `Dockerfile`).

### Step 1 – Build the image

```
docker build -t program-1 .
```

- `-t program-1` names the image `program-1`.
- The `.` at the end means "use the current folder as the build context". Do not leave it out.

Confirm the image exists:

```
docker images
```

You should see `program-1` with tag `latest`.

### Step 2 – Run the container

```
docker run -d -p 5000:5000 --name flask-container program-1
```

- `-d` runs the container in the background.
- `-p 5000:5000` maps port 5000 on your machine to port 5000 in the container.
- `--name flask-container` gives the container a name you can use in later commands.

Check that it is running:

```
docker ps
```

The `STATUS` column should say `Up ...`.

### Step 3 – Test it

Open a browser and go to:

```
http://localhost:5000
```

Expected output:

```
Hello, Docker!
```

### Step 4 – Stop and remove the container

```
docker container stop flask-container
docker container rm flask-container
```

You can also use the container ID shown by `docker container ls -a` instead of the name.

### Step 5 – Delete the image

```
docker image rm program-1
```

## Troubleshooting

| Problem | Fix |
|---|---|
| `error during connect` / `Cannot connect to the Docker daemon` | Docker Desktop is not running. Start it and try again. |
| `port is already allocated` | Port 5000 is in use. Run on another host port, e.g. `docker run -d -p 5001:5000 --name flask-container program-1`, then open `http://localhost:5001`. |
| `The container name "/flask-container" is already in use` | A container with that name already exists. Remove it with `docker rm -f flask-container`, then run again. |
| Page does not load | Check the logs with `docker logs flask-container`. |

## Expected Outcomes

- A Docker image is created from a user-defined Dockerfile.
- The container runs with all dependencies installed.
- The same image runs the same way on any machine that has Docker.
