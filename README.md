# My Raspberry Pi 5 HomeLab Setup

Welcome to the official documentation of my personal HomeLab. This guide tracks my entire deployment journey, starting from scratch (hardware selection and OS flashing) all the way to hosting cloud services and custom web applications under my domain `cottihomelab.uk`.

---

## Phase 1: Hardware & OS Selection

### The Hardware
* **SBC:** Raspberry Pi 5 (8GB RAM)
* **Storage:** High-speed MicroSD / NVMe SSD / SSD via USB 3.1 / USB Drive (offering flexible options for performance and durability)
* **Power Supply:** Official Raspberry Pi 5 27W USB-C Power Supply (crucial to prevent voltage drops under heavy Docker workloads)

### The Operating System: Why Raspberry Pi OS (64-bit) Lite?
For this setup, I chose **Raspberry Pi OS 64-bit (Lite/Headless)**. Here is why:

1. **64-bit Architecture:** Essential to fully utilize the 8GB of RAM on the Raspberry Pi 5 and ensure native compatibility with modern Docker images (ARM64).
2. **Lite Version (No GUI):** A server doesn't need a desktop environment. Stripping away the graphical interface saves hundreds of megabytes of RAM and CPU cycles, dedicated entirely to running containers.
3. **Debian Stability:** Being built on top of Debian Linux guarantees rock-solid stability and long-term security updates.

---

## Phase 2: Flashing & Initial Configuration (Headless)

The entire configuration was performed in **Headless mode** (without ever connecting the Raspberry Pi to a monitor or keyboard), managing everything remotely over the local network via **SSH**.

### 1. Flashing with Raspberry Pi Imager
Using the official *Raspberry Pi Imager* software on my main PC, I customized the advanced OS customization settings before flashing the drive:
* Enabled the **SSH** server (configured with password authentication).
* Created the main user account (`cotti`) and its secure password.
* Pre-configured the home network credentials (Wi-Fi/Ethernet).

### 2. First Boot & System Update
Once the Raspberry Pi booted up and registered on the local network, its IP address (`192.168.x.y`) was verified either by running `hostname -I` directly on the machine or by using the `ping` command from another local device. 

With the network IP confirmed, I established the first remote connection via the terminal:

```bash
ping raspberrypi.local
ssh user@ip_address(EX 192.168.1.4)
