ustomer Loyalty Program Code Documentation

This document provides an overview of the C++ code for a customer loyalty program implemented for a small bookstore.

## Table of Contents

1. [Introduction](#introduction)
2. [Code Structure](#code-structure)
3. [Data Structures](#data-structures)
4. [Thread Management](#thread-management)
5. [Camera Configuration](#camera-configuration)
6. [Mutex Usage](#mutex-usage)
7. [Program Signals](#program-signals)
8. [Camera Activation](#camera-activation)

---

### Introduction <a name="introduction"></a>

The code is designed to implement a customer loyalty program for a small bookstore. It appears to involve video streaming and camera management, but further details are needed for a complete understanding.

### Code Structure <a name="code-structure"></a>

The code is written in C++ and includes libraries for OpenCV and FFmpeg. It is organized into several sections, including camera data, video buffering, mutex usage, and thread management.

### Data Structures <a name="data-structures"></a>

#### `struct cameradata`

- Stores camera parameters such as URL, username, password, and port.
- Includes a thread ID for system allocation.

#### `struct videobuffer`

- Represents a circular queue for storing video data (AVPacket).
- Maintains `front` and `rear` indices for data management.

#### `struct parameterpack`

- Used for passing parameters to decoding processes.
- Includes a process ID (`pid`), a stringstream pointer (`id`), a format context pointer (`fmtCtx`), and a video stream index (`videoStreamIndex`).

### Thread Management <a name="thread-management"></a>

The code manages multiple threads for camera activation, decoding, and more. Thread management includes activation flags, thread IDs, and signals.

### Camera Configuration <a name="camera-configuration"></a>

Camera configurations are stored in the `cameradata` struct, including URLs, usernames, passwords, and ports. The code appears to support multiple cameras.

### Mutex Usage <a name="mutex-usage"></a>

Mutexes (`std::mutex`) are used extensively to protect shared data between threads. Different mutexes are employed for different purposes, including protecting data between decoding and accessing processes.

### Program Signals <a name="program-signals"></a>

Various signals and flags are used throughout the program to manage thread activation, data availability, and process termination.

### Camera Activation <a name="camera-activation"></a>

\# `cameradd` Function Documentation

This documentation explains the `cameradd` function, which is responsible for adding cameras to the program during runtime. The function accepts camera data, the number of cameras to add, and whether it should be blocked.

\## Function Signature

\```cpp
void cameradd(struct cameradata* carema, int number, int isblocked);
\```

\## Parameters

- `carema`: An array of `struct cameradata` containing camera configuration data.
- `number`: The number of cameras to add. Must be less than or equal to `NUMBER_OF_CAMERA`.
- `isblocked`: Indicates whether the function should block until the cameras are successfully added.

\## Purpose

The `cameradd` function is designed to dynamically add cameras to the program. It copies camera configuration data into the `carema_activation_data` array, marks the activation flags, and sets a signal to indicate that cameras are being set up. If the function is not blocked (`isblocked == 0`), it will wait for the cameras to connect successfully.

\## Steps

1. **Copy Camera Configuration Data**:
   - The function copies camera configuration data (URL, port, password, and username) from the `carema` array into the `carema_activation_data` array for further processing.

2. **Mark Activation Flags**:
   - It sets the `activation_flag` for each camera to `1`, indicating that the cameras are being activated.

3. **Set Up Signal**:
   - The function sets the `carema_setsup_signal` variable to `1` to signal that cameras are being set up.

4. **Check Blocking**:
   - If `isblocked` is `1`, the function returns immediately without waiting. This is useful when you want to add cameras without blocking the program's execution.

5. **Wait for Camera Connection (Optional)**:
   - If `isblocked` is `0`, the function enters a loop and waits for the cameras to connect successfully. It checks the `activation_flag` for each camera.
   - If all cameras are successfully connected (`activation_flag[y] == 2`), it resets the flags to `0` and prints a success message.
   - If there are issues with camera setup, it prints an error message.

\## Note

- The function uses the `av_usleep` function to introduce delays.
- The waiting loop is designed to wait for a short period (3 seconds in this case) and can be adjusted as needed.

# `cameradisplay` Function Documentation

This documentation explains the `cameradisplay` function, which handles the decoding, transcoding, and display of video streams. It operates within a loop, decoding video frames from a circular buffer, performing transcoding if needed, and updating the video display slot.

## Function Signature

void cameradisplay(struct parameterpack pkg);

## Parameters

- pkg: A structure containing parameters required for video processing, including the process ID (pid), a stringstream pointer (id), the format context (fmtCtx), and the video stream index (videoStreamIndex).

## Purpose

The cameradisplay function is responsible for processing video frames from a circular buffer. It decodes video data, performs transcoding to a specified format, and updates the video display slot. This function operates within a loop, continuously processing video data.

## Steps

1. Initialize Variables:
   - Initialize various variables, including codec parameters, codec context, frames, and buffers required for video processing.

2. Video Processing Loop:
   - Enter a continuous loop for video processing.
   - Check the circular buffer for available video frames (gbbuffer[pid]).
   - If a frame is available, decode it using the codec.
   - If decoding is successful, perform transcoding if necessary and update the video display slot (player[pid]).

3. Transcoding:
   - Transcoding is performed when necessary to convert the video to a specified format (e.g., BGR24).

4. Update Video Display Slot:
   - Update the video display slot with the processed video frame.

5. Thread Synchronization:
   - Implement mutex locks (mtx[pid], mtxplay[pid]) to ensure thread safety when accessing shared resources.

6. Delay and Continue:
   - Introduce a delay (10 milliseconds) if there are no video frames to process, enhancing efficiency.

7. Cleanup:
   - Free allocated memory and release resources.

## Note

- This function processes video data from a circular buffer (gbbuffer[pid]) and updates the video display slot (player[pid]).
- It uses FFmpeg libraries for decoding and transcoding video data.
- Thread synchronization is implemented using mutex locks to ensure data integrity and prevent conflicts.

Tmerafetcher` Function Documentation

This documentation explains the `camerafetcher` function, which is responsible for fetching video streams from cameras via RTSP. It establishes a connection to the camera, reads video frames, and stores them in a circular buffer for further processing.

## Function Signature

int camerafetcher(struct cameradata ca);

## Parameters

- ca: A structure containing camera configuration data, including the URL, username, password, port, and thread ID.

## Purpose

The camerafetcher function is tasked with accessing camera streams, fetching video frames, and storing them in a circular buffer. It handles the setup of the RTSP connection, reads video frames, and manages thread synchronization.

## Steps

1. Initialize Variables:
   - Initialize various variables, including the RTSP URL, AVFormatContext, and options required for RTSP connection.

2. RTSP Connection:
   - Attempt to establish an RTSP connection to the camera using FFmpeg libraries.
   - Implement retry logic for connection attempts, with a maximum of three retries.
   - If the connection fails, the function sets an activation flag and terminates the thread.

3. Initialize Circular Buffer:
   - Initialize a circular buffer (gbbuffer[ca.threadid]) to store video frames.
   - Initialize AVPacket structures within the buffer.

4. Find Video Stream:
   - Find the video stream index within the format context.

5. Thread Creation:
   - Create a new thread (player) to handle video display and processing (using the cameradisplay function).

6. Video Fetching Loop:
   - Continuously read video frames from the camera and store them in the circular buffer.
   - Implement thread-safe access to the circular buffer using mutex locks.

7. Cleanup:
   - Release resources and close the RTSP connection when done.

## Note

- This function fetches video frames from an RTSP camera stream.
- It uses FFmpeg libraries for RTSP handling and AVPacket management.
- Thread synchronization is implemented using mutex locks (mtx[ca.threadid]) for accessing the circular buffer.
- The circular buffer is used to temporarily store video frames before processing.

gmgviewer_start Function Documentation

This documentation explains the gmgviewer_start function, which is responsible for managing the creation of decoding and transcoding threads for video processing. It initializes the necessary FFmpeg libraries and starts video processing threads for camera streams.

Function Signature

int gmgviewer_start(int thid, std::string url, std::string name, std::string pass, std::string port);

Parameters

- thid: An integer representing the thread ID to be assigned to the camera processing thread.
- url: A string containing the URL of the camera stream.
- name: A string containing the username for authentication.
- pass: A string containing the password for authentication.
- port: A string containing the port for the camera stream connection.

Purpose

The gmgviewer_start function serves as a management tool for creating and starting video processing threads. It initializes the required FFmpeg libraries, sets up camera data, and starts the camerafetcher thread for processing camera streams.

Steps

1. Initialize Variables:
   - Initialize thread management variables, including an array of threads (the_list) and a thread count (threadcount).

2. FFmpeg Library Initialization:
   - If threadcount is -1, initialize FFmpeg libraries:
     - Call av_register_all() to register all available codecs and formats.
     - Call avformat_network_init() to initialize network components of FFmpeg.

3. Camera Data Setup:
   - Prepare camera configuration data by populating a cameradata structure (ca) with the provided URL, username, password, port, and thread ID.

4. Thread Creation:
   - Create a new thread (camerafetcher) to handle video stream fetching and processing.
   - Increment the threadcount to keep track of active threads.

5. Unlock Mutex:
   - Release the mutex lock (mtxgb2) to allow concurrent thread creation.

6. Return Thread ID:
   - Return the assigned thread ID for reference.

Note

- This function manages the creation of video processing threads for camera streams.
- It initializes necessary FFmpeg libraries and creates threads for fetching and processing camera streams.
- Thread management and synchronization are implemented using mutex locks (mtxgb2).
- The function returns the assigned thread ID for reference.

## `int start_algorithm(struct cameradata arg)`

This function represents the core algorithmic processing for each camera stream.

### Parameters

- `arg`: A structure (`struct cameradata`) containing information about the camera, including its URL, username, password, and port.

### Functionality

1. This function starts by receiving camera data as input and initializing various variables for processing.

2. It continuously loops to process frames from the camera stream.

3. If the `terminate_flag` for the camera thread is set to 1, the thread is terminated and joined.

4. It checks if there's a new frame available from the camera stream, and if so, it preprocesses the image.

5. Depending on the value of `SQL::model_flag`, it either performs face recognition on the incoming frame or waits for a new face to be registered.

6. When `SQL::model_flag` is 0 (face recognition mode):
   - It preprocesses the image.
   - Runs inference on the image using a TensorRT context (`context`) and a deep learning model.
   - Matches the detected face features with stored face features to identify a person.
   - If a matching person is found, it triggers an event and saves the detected image.

7. When `SQL::model_flag` is 1 (new face registration mode):
   - It preprocesses the image.
   - Draws bounding boxes around detected faces.
   - Displays the image for manual selection of the face to register.
   - Registers the selected face by storing its features and ID in a SQLite database.

8. The function also measures and logs the time taken for different steps of processing.

### Return Value

- The function does not return a value.

### Usage

```cpp
struct cameradata camera_info;
camera_info.url = "rtsp://camera_url";
camera_info.username = "username";
camera_info.password = "password";
camera_info.port = "port_number";

start_algorithm(camera_info);
## `int main(int argc, char* argv[])`

The `main` function is the entry point of your application. It initializes various components, manages camera threads, and handles real-time image processing.

### Parameters

- `argc`: The number of command-line arguments.
- `argv`: An array of command-line argument strings.

### Initialization

1. Sets the `path` and `path1` variables to specific directory paths.

2. Initializes various global variables, such as `terminate_flag`, `activation_flag`, `threadmap`, and others, to control and manage camera threads.

3. Retrieves a list of filenames in a directory specified by `path` using the `GetAllFilenames` function.

4. Starts a server asynchronously using `std::async` to run the `Service::start` function.

5. Builds TensorRT execution contexts (`context0` and `context`) by loading corresponding engine files (`filepath_0` and `filepath`) for deep learning models.

6. Sets the CUDA device using `cudaSetDevice(DEVICE)`.

7. Initializes HTTP client and SQLite database connections using `HttpClient::init()` and `SQL::init()`.

### Camera Initialization and Testing (Sample Code)

- Initializes an array of `struct cameradata` objects (`arg`) with camera information.

- Calls the `cameradd` function to initialize and activate camera threads.

### Main Loop

1. The main loop performs the following tasks:
   - Pauses execution for a short duration (`av_usleep`) to control the frame rate.

   - Checks and joins terminated camera threads.

   - Displays frames from active camera threads using OpenCV (`imshow`) and waits for a key press to update the display.

   - Monitors signals to set up new camera threads (`carema_setsup_signal`) and activates them.

2. Inside the loop, camera threads are created and managed dynamically based on activation signals.

### Return Value

- The `main` function returns an integer, typically `0`, to indicate successful execution.

### Usage

```cpp
int main(int argc, char* argv[]) {
    // Initialize variables and components
    // Set paths, initialize global variables, create server, load models, etc.

    // Initialize and activate camera threads
    struct cameradata arg[NUMBER_OF_CAMERA];
    // Populate camera information in 'arg'
    cameradd(arg, NUMBER_OF_CAMERA, 0);

    // Main loop for monitoring and managing camera threads
    while (1) {
        // Perform various tasks, including camera thread management and frame display
    }

    return 0;
}
This documentation provides an initial overview of the code's structure and key components. Further details and functionality descriptions would be needed for a more comprehensive understanding.
