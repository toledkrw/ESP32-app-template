# ESP32 Project Setup with Docker and Devcontainers

## Table of Contents
1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
    - [ESP32 Driver Installation](#esp32-driver-installation)
3. [Setting Up the Environment](#3-setting-up-the-environment)
    - [WSL (Windows Subsystem for Linux)](#wsl)
    - [Docker](#docker)
    - [VSCode](#vscode)
        - [ESP-IDF Extension](#-esp-idf-extension)
        - [Dev Containers Extension](#-devcontainers-extension)
4. [USB Forwarding](#4-usb-forwarding)
    - [Windows to WSL](#windows-to-wsl)
    - [WSL to Docker Container](#wsl-to-docker-container)
    - [Undoing USB Forwarding](#undoing-usb-forwarding)
5. [Cloning the Project](#5-cloning-the-project)
6. [Project Structure](#6-project-structure)
7. [Build and Deploy](#7-build-and-deploy)
8. [Tips and Troubleshooting](#8-tips-and-troubleshooting)
9. [References](#9-references)

---

## 1. Introduction
This guide aims to detail the process of setting up a modern and reproducible development environment for ESP32 projects using tools like Docker, Devcontainers, and WSL. Instead of focusing on a specific project, the goal is to standardize and simplify environment preparation, making it easier for any developer to start, collaborate, and maintain ESP32 projects regardless of the operating system used. Here you will find step-by-step instructions for installing drivers, configuring WSL, Docker, VSCode, and essential extensions, as well as tips for USB forwarding and troubleshooting common issues.

## 2. Prerequisites
- Basic knowledge of electrical and electronics
- Programming knowledge
    - C & C++
    - Shell
    - Python
- ESP32
- A COMPUTER 🖥️
- USB cable
- Internet connection

### ESP32 Driver Installation
For your computer to correctly recognize the ESP32, you may need to install the USB to UART driver, especially for boards based on the CP210x (Silicon Labs) chip.

### Windows
1. **Identify the device:**  
    When connecting the ESP32, if it is not recognized, it will appear in Device Manager as `CPxxxx USB to UART Bridge Controller` (e.g., `CP2102 USB to UART Bridge Controller`).
2. **Download the driver:**  
    Go to the official Silicon Labs website:  
    [https://www.silabs.com/developer-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads](https://www.silabs.com/developer-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads)
3. **Install the driver:**  
    - Download the installer for your operating system (Windows 10/11, 64 or 32 bits).
    - Run the installer and follow the on-screen instructions.
4. **Manually update the driver (if necessary):**  
    - In Device Manager, right-click the `CPxxxx USB to UART Bridge Controller` device and select **Update driver**.
    - Choose **Browse my computer for drivers** and point to the folder where the driver was extracted/installed.
5. **Verify the installation:**  
    After installation, the device should appear as a COM port (e.g., `COM3`) and be ready for use.

### Linux
On most modern Linux distributions, the CP210x driver is already included in the kernel. To check:

1. **Connect the ESP32 via USB.**
2. **Check if the device was recognized:**  
    Run in the terminal:
    ```bash
    dmesg | grep tty
    ```
    Look for something like `/dev/ttyUSB0` or `/dev/ttyACM0`.
3. **If not recognized:**  
    - Make sure the module is loaded:
      ```bash
      lsmod | grep cp210x
      ```
    - If not, load it manually:
      ```bash
      sudo modprobe cp210x
      ```
    - If it still doesn't work, check if your user belongs to the `dialout` group:
      ```bash
      sudo usermod -aG dialout $USER
      ```
      Then, restart your session.
4. **Reference:**  
    [Silicon Labs Drivers - Documentation](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)

With the driver installed, the ESP32 will be ready for serial communication on both Windows and Linux.


## 3. Setting Up the Environment

### WSL
To manually install WSL (Windows Subsystem for Linux), follow the steps below:

1. **Enable the WSL feature:**
    Open PowerShell as administrator and run:
    ```powershell
    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
    ```
2. **Enable the Virtual Machine Platform:**
    Still in PowerShell, run:
    ```powershell
    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
    ```
3. **Restart your computer.**
4. **Install the WSL kernel:**
    Download and run the updated kernel installer at:  
    [https://aka.ms/wsl2kernel](https://aka.ms/wsl2kernel)
5. **Set WSL 2 as default (optional but recommended):**
    In PowerShell, run:
    ```powershell
    wsl --set-default-version 2
    ```
6. **Install a Linux distribution:**
    Go to the Microsoft Store, search for "Ubuntu" (or another distribution of your choice), and install it.
7. **Set up the distribution:**
    After installation, open the distribution from the start menu and follow the instructions to create a user and password.

**Reference:**  
[Official WSL Manual Installation Guide](https://learn.microsoft.com/en-us/windows/wsl/install-manual)

### Docker
To install Docker, follow the steps below according to your operating system:

#### Windows
1. **Go to the official website:**  
    [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. **Download the Docker Desktop installer.**
3. **Run the installer:**  
    Follow the on-screen instructions to complete the installation.
4. **Restart your computer if prompted.**
5. **Verify the installation:**  
    Open the terminal (cmd, PowerShell, or WSL) and run:
    ```bash
    docker --version
    ```
    The command should return the installed Docker version.

#### Linux (Ubuntu)
1. **Update packages:**
    ```bash
    sudo apt update
    ```
2. **Install dependencies:**
    ```bash
    sudo apt install apt-transport-https ca-certificates curl software-properties-common
    ```
3. **Add the official Docker repository:**
    ```bash
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
4. **Install Docker:**
    ```bash
    sudo apt update
    sudo apt install docker-ce
    ```
5. **Verify the installation:**
    ```bash
    docker --version
    ```
6. **(Optional) Add your user to the docker group:**
    ```bash
    sudo usermod -aG docker $USER
    ```
    Then, restart your session to apply the change.

#### macOS
1. **Go to the official website:**  
    [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. **Download the Docker Desktop installer for Mac.**
3. **Open the .dmg file and drag Docker to the Applications folder.**
4. **Open Docker Desktop and follow the setup instructions.**
5. **Verify the installation in the terminal:**
    ```bash
    docker --version
    ```

**Reference:**  
[Official Docker Documentation](https://docs.docker.com/get-docker/)

### VSCode
To install Visual Studio Code (VSCode), follow the steps below:

1. **Go to the official website:**  
    [https://code.visualstudio.com/](https://code.visualstudio.com/)
2. **Download the installer:**  
    Choose the appropriate version for your operating system (Windows, macOS, or Linux).
3. **Run the installer:**  
    Follow the on-screen instructions to complete the installation.
4. **(Optional) Install VSCode via command line:**  
    - **Windows:**  
      ```powershell
      winget install Microsoft.VisualStudioCode
      ```
    - **Ubuntu/Linux:**  
      ```bash
      sudo snap install --classic code
      ```
5. **Verify the installation:**  
    Open VSCode and confirm it is working correctly.

It is recommended to also install the "Remote - Containers" extension to work with Devcontainers.

#### - ESP-IDF Extension
To install the ESP-IDF extension in VSCode, follow these steps:

1. **Open VSCode.**
2. **Go to the extensions tab:** Click the extensions icon in the left sidebar or press `Ctrl+Shift+X`.
3. **Search for "ESP-IDF":** In the search field, type `ESP-IDF`. Or use the direct link: [https://marketplace.visualstudio.com/items?itemName=espressif.esp-idf-extension](https://marketplace.visualstudio.com/items?itemName=espressif.esp-idf-extension)
4. **Select the "Espressif IDF" extension:** Click on the extension developed by Espressif Systems.
5. **Click "Install".**

After installation, follow the instructions shown to set up the ESP-IDF environment, including installing dependencies and configuring Python if necessary.

This extension makes it easier to develop, build, flash, and debug ESP32 projects directly from VSCode.
You will need to install the IDF version you will use in a "sys path", but when developing through a devcontainer, this will be done automatically.


#### - Dev Containers Extension
To install the "Dev Containers" extension in VSCode, follow these steps:

1. **Open VSCode.**
2. **Go to the extensions tab:** Click the extensions icon in the left sidebar or press `Ctrl+Shift+X`.
3. **Search for "Dev Containers":** In the search field, type `Dev Containers`. Or use the direct link: [https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
4. **Select the "Dev Containers" extension (Microsoft):** Click on the extension developed by Microsoft.
5. **Click "Install".**

After installation, you will be able to open folders or projects inside development containers, making it easier to set up isolated and reproducible environments for your ESP32 project.

## 4. USB Forwarding

### Windows to WSL
For the ESP32 connected via USB to be accessible inside WSL, you need to "bind" the device using the `usbipd` utility. Follow the steps below:

> Before listing devices, install the `usbipd` utility on Windows if you haven't already. Run in PowerShell:
> ```powershell
> winget install usbipd
> ```
1. **List available USB devices in Windows:**
    In PowerShell (outside WSL), run:
    ```powershell
    usbipd list
    ```
    Look for the device corresponding to the ESP32 (usually identified as "CP210x" or "Silicon Labs").
2. **Note the BUSID and VID:PID of the device**  
    Example: `1-3` and `A0B1:0A1B`).
3. **Attach the device to WSL:**
    Still in PowerShell, run:
    ```powershell
    usbipd attach --busid <ID> --wsl
    ```
    Replace `<ID>` with the identifier noted earlier.
4. **Check in WSL:**
    In the WSL terminal, run and check if the device appears as a USB with the message:
    ```bash
    dmesg | tail -n 10
    "usb 1-1: cp210x converter now attached to ttyUSB0"
    ```
5. **Manually assign the driver if necessary**
    If you don't see the previous message or don't have any ```ttyUSB``` device:
    ```bash
    ls /dev/ttyUSB*
    ```
    You will need to manually assign the driver to the device, then mount it to a USB node. Remember your "VID:PID" and replace them in the following command:
    ```bash
    modprobe cp210x
    echo <VID> <PID> | sudo tee /sys/bus/usb-serial/drivers/cp210x/new_id
    ```
    Check again if the device was mounted to a ```ttyUSB```:
    ```bash
    dmesg | tail -n 10
    "usb 1-1: cp210x converter now attached to ttyUSB0"
    ```    

**Reference:**  
[Connect USB devices to WSL - Microsoft Documentation](https://learn.microsoft.com/en-us/windows/wsl/connect-usb#attach-a-usb-device)


### WSL to Docker Container
When you use `usbipd` to bind the USB device to your WSL instance (e.g., `Ubuntu` or `docker-desktop`), the USB device becomes visible to the WSL kernel. Since Docker Desktop for Windows runs Linux containers inside WSL 2, containers can access USB devices exposed by WSL, as long as the device is explicitly mapped to the container using the `--device` option in `devcontainer.json`. That is, the forwarding done by `usbipd` makes the device available to WSL, but each container still needs permission to access the device (e.g., `/dev/ttyUSB0`) via configuration in `devcontainer.json`. No additional drivers are needed inside the container if the device is already functional in WSL, but device mapping is mandatory for container access.
```json
"runArgs": [
        "--device=/dev/ttyUSB0:/dev/ttyUSB0",
        "--privileged"
    ]
```

### Undoing USB Forwarding

To undo the forwarding of a USB device connected to WSL, use the command:
```powershell
usbipd detach --busid <ID>
```
Replace `<ID>` with the USB bus identifier of the device that was previously forwarded.
This command disconnects the USB device from the WSL environment, making it available again to Windows or other systems.

> Note that you may need to repeat the USB forwarding and node mounting process if you change your device's USB port, as its ```BUSID``` will change.

## 5. Cloning the Project
<!-- TODO -->

## 6. Project Structure
The structure of this project is simple and follows the recommended pattern for ESP32 projects with CMake:

```
├── CMakeLists.txt           # Main CMake configuration file for the project
├── main
│   ├── CMakeLists.txt       # CMake configuration specific to the main source code
│   └── main.c               # Main application source code
└── README.md                # This documentation file
```

- **CMakeLists.txt**: Root CMake configuration file, responsible for defining the project's build rules.
- **main/**: Directory containing the main application source code, including its own CMakeLists.txt and the `main.c` file.
- **README.md**: Project documentation, with usage instructions, configuration, and relevant information.

This organization makes the project easier to maintain, scale, and understand, following best practices for CMake-based ESP32 projects.

## 7. Build and Deploy
<!-- TODO -->

## 8. Tips and Troubleshooting
<!-- TODO -->

## 9. References

- [Official ESP-IDF Documentation](https://docs.espressif.com/projects/esp-idf/en/latest/)
- [Manual WSL Installation Guide (Microsoft)](https://learn.microsoft.com/en-us/windows/wsl/install-manual)
- [Connect USB devices to WSL (Microsoft)](https://learn.microsoft.com/en-us/windows/wsl/connect-usb)
- [USB to UART Bridge Drivers (Silicon Labs)](https://www.silabs.com/developer-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads)
- [Official Docker Documentation](https://docs.docker.com/get-docker/)
- [Visual Studio Code - Official Site](https://code.visualstudio.com/)
- [ESP-IDF Extension for VSCode](https://marketplace.visualstudio.com/items?itemName=espressif.esp-idf-extension)
- [Dev Containers Extension for VSCode](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- [Awesome ESP32 Repository (resources and projects)](https://github.com/agucova/awesome-esp32)
- [ESP32 Community on GitHub](https://github.com/espressif/esp-idf)
- [Docker Desktop for Windows/macOS](https://www.docker.com/products/docker-desktop/)