Containerized 3D Gaussian Splatting on Windows 11
This project provides a Docker setup to run the official implementation of 3D Gaussian Splatting for Real-Time Radiance Field Rendering. Using Docker simplifies dependency management and ensures a consistent, reproducible environment for training and rendering, especially when working on Windows.

Prerequisites
Before you begin, ensure you have the following installed and configured on your Windows 11 machine:

Docker Desktop for Windows: Installation Guide. Make sure it is configured to use the WSL 2 backend, which is the default for modern installations.

NVIDIA GPU: A CUDA-enabled NVIDIA GPU is required.

NVIDIA GPU Drivers: Install the latest drivers for your GPU on your Windows host. Docker Desktop will automatically handle passing the GPU into the Linux container.

1. Building the Docker Image
First, build the Docker image using the provided Dockerfile. This will install all dependencies, including FFmpeg and COLMAP for video processing, inside a Linux environment.

Step 1.1: Clean Your Docker Environment (Important First Step)
If you have encountered errors like cannot execute binary file, it's crucial to start from a clean slate. Run the following command in PowerShell to remove all stopped containers, unused networks, and dangling images.

docker system prune -a -f

Step 1.2: Build the Image
Open a terminal (like PowerShell) in the directory containing the Dockerfile and run the build command.

docker build --platform linux/amd64 --no-cache -t gaussian-splatting .

Important:

--platform linux/amd64: This ensures the image is built for the correct CPU architecture (AMD64).

--no-cache: This forces Docker to rebuild the image from scratch, preventing issues with corrupted or incorrect cached layers.

2. Processing a Video Input
This section explains how to convert a video file into a dataset suitable for training.

2.1. Prepare Directory Structure
On your Windows machine, create a directory for your project. Inside, create a data directory where you will place your video. We will use a scene name, for example truck, to organize the files.

Crucially, the folder containing the extracted images must be named input.

For example, you could create the following structure in your Documents folder:

C:\Users\YourUser\Documents\gaussian-project\
└── data/
    └── truck/
        ├── input.mp4      # Your source video
        └── input/         # FFMpeg will save frames here

Place your video (e.g., input.mp4) inside the data\truck\ directory and create an empty input subdirectory.

2.2. Convert Video to Images
Run the container to execute ffmpeg. This command will extract the frames into the input folder.

docker run --platform linux/amd64 --gpus all -it --rm `
  -v C:\Users\YourUser\Documents\gaussian-project\data:/workspace/gaussian-splatting/data `
  gaussian-splatting `
  ffmpeg -i data/truck/input.mp4 -qscale:v 1 -qmin 1 -vf "fps=2" data/truck/input/%04d.jpg

Note on Paths: The -v flag maps a directory from your Windows host to a directory inside the container. Replace C:\Users\YourUser\Documents\gaussian-project\ with the absolute path to your project directory.

2.3. Process Images with COLMAP
Next, run COLMAP on the extracted images to generate camera poses. We have added the --colmap_camera_model OPENCV_FISHEYE flag to make the process more robust.

docker run --platform linux/amd64 --gpus all -it --rm `
  -v C:\Users\YourUser\Documents\gaussian-project\data:/workspace/gaussian-splatting/data `
  gaussian-splatting `
  python3.8 convert.py -s data/truck --colmap_camera_model OPENCV_FISHEYE

After this step, your data\truck directory will contain a sparse folder with the COLMAP reconstruction, making it ready for training.

3. Training the Model
Once your data has been processed, you can start the training process. This command will save the model output to a new output directory within your project folder.

Example Training Command
docker run --platform linux/amd64 --gpus all -it --rm `
  -v C:\Users\YourUser\Documents\gaussian-project\data:/workspace/gaussian-splatting/data `
  -v C:\Users\YourUser\Documents\gaussian-project\output:/workspace/gaussian-splatting/output `
  gaussian-splatting `
  python3.8 train.py -s data/truck -m output/truck_model
