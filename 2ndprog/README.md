# Program 2 – Develop a Multi-Stage Dockerfile for Container Orchestration

This program builds a small **Node.js / Express** web app using a **multi-stage Dockerfile**:

- **Stage 1 (builder):** installs dependencies and runs the build (`npm run build` copies `src/` into `dist/`).
- **Stage 2 (production):** starts from a fresh `node:20-alpine` image and copies in only what is needed to run the app (`package.json`, `package-lock.json`, `dist/`, `node_modules/`).

This keeps the final image smaller and cleaner. The app returns `Hello from multi-stage Docker!` at `http://localhost:3000`.

## Project Structure

```
2ndprog/
├── Dockerfile          # Multi-stage build instructions
├── .dockerignore       # Keeps local node_modules/dist out of the image
├── package.json        # Project info, scripts and dependencies
├── package-lock.json   # Created by "npm install"
├── src/
│   └── index.js        # Express application
├── node_modules/       # Created by "npm install"
└── README.md
```

## Prerequisites

| Requirement | Notes |
|---|---|
| **Docker** | Docker Desktop (Windows/macOS) or Docker Engine (Linux). On Windows, Docker Desktop needs WSL 2 enabled. |
| **Node.js and npm** | Node.js 18 or newer (20 LTS recommended). Needed for Step 1 to create `package-lock.json`. Download from https://nodejs.org. |
| **Internet connection** | Needed to download npm packages and the `node:20-alpine` base image. |
| **Free port 3000** | Nothing else should be using port 3000 on your machine. |
| A terminal | PowerShell, Command Prompt, Git Bash or a Linux/macOS terminal. |

Check that the tools are installed:

```
docker --version
docker info
node --version
npm --version
```

If `docker info` shows an error, start Docker Desktop and wait until it reports that Docker is running.

## How to Execute

Open a terminal **inside the `2ndprog` folder** (the folder that contains the `Dockerfile`).

### Step 1 – Install Node modules

```
npm install
```

This creates `node_modules/` and `package-lock.json`.

> Do **not** run `npm run build` on Windows PowerShell or Command Prompt. It uses Linux commands (`mkdir -p`, `cp`) and is meant to run inside the Docker container, which happens automatically in Step 2.

### Step 2 – Build the image

```
docker build -t program-2 .
```

- `-t program-2` names the image `program-2`.
- The `.` at the end means "use the current folder as the build context". Do not leave it out.

In the output you will see steps for both stages, labelled `[builder ...]` and `[stage-1 ...]`.

Confirm the image exists:

```
docker images
```

You should see `program-2` with tag `latest`.

### Step 3 – Run the container

```
docker run -d -p 3000:3000 --name node-container program-2
```

- `-d` runs the container in the background.
- `-p 3000:3000` maps port 3000 on your machine to port 3000 in the container.
- `--name node-container` gives the container a name you can use in later commands.

Check that it is running:

```
docker ps
docker logs node-container
```

The logs should show `Server running on port 3000`.

### Step 4 – Test it

Open a browser and go to:

```
http://localhost:3000
```

Expected output:

```
Hello from multi-stage Docker!
```

### Step 5 – Stop and remove the container

```
docker container stop node-container
docker container rm node-container
```

You can also use the container ID shown by `docker container ls -a` instead of the name.

### Step 6 – Delete the image

```
docker image rm program-2
```

## Troubleshooting

| Problem | Fix |
|---|---|
| `error during connect` / `Cannot connect to the Docker daemon` | Docker Desktop is not running. Start it and try again. |
| `'npm' is not recognized` | Node.js is not installed or not on PATH. Install it from https://nodejs.org and open a new terminal. |
| `port is already allocated` | Port 3000 is in use. Run on another host port, e.g. `docker run -d -p 3001:3000 --name node-container program-2`, then open `http://localhost:3001`. |
| `The container name "/node-container" is already in use` | A container with that name already exists. Remove it with `docker rm -f node-container`, then run again. |
| Page does not load | Check the logs with `docker logs node-container`. |

## Expected Outcomes

- A multi-stage Dockerfile separates the build environment from the runtime image.
- The final image contains only the files needed to run the app.
- The containerised app runs the same way on any machine that has Docker.
