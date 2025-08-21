# Nextcloud VPN Tunnel

“Nextcloud-VPN-Tunnel” is a proof-of-concept project to create a store-and-forward VPN tunnel over Nextcloud file sync. This project explores the possibility of using Nextcloud's file synchronization mechanism as a transport layer for VPN-like traffic, offering a high-latency but potentially stealthy way to bypass network restrictions.

## How it Works

The core idea is to encapsulate network traffic into files, sync them via Nextcloud, and then reconstruct the traffic on the other end.

1.  **Client-Side:** A client application captures packets from a local VPN interface (e.g., TUN/TAP). These packets are then encrypted, encoded, and written as small file chunks into a designated Nextcloud sync folder.
2.  **Nextcloud Sync:** The Nextcloud client syncs these file chunks to the Nextcloud server.
3.  **Server-Side:** A server application, running on the same machine as the Nextcloud server, detects these new file chunks. It decrypts and decodes them, reconstructing the original network packets. These packets are then injected into the server's VPN interface and forwarded to their destination.
4.  **Return Traffic:** The process is reversed for traffic from the server to the client.

This creates a "store-and-forward" tunnel, where data is stored in files and forwarded by the Nextcloud sync mechanism.

## Project Status

This project is currently a **proof-of-concept** and is **not ready for production use**. The core components are in the early stages of development, and the Python scripts are placeholders.

## Components

*   **Client:** A Python script (`client/client.py`) that will be responsible for capturing, encrypting, and writing traffic to files.
*   **Server:** A Python script (`server/server.py`) that will be responsible for reading, decrypting, and injecting traffic from files.
*   **Configuration:** A JSON file (`config/config_example.json`) to store settings for the Nextcloud connection and encryption.
*   **Documentation:** High-level design is documented in `docs/architecture.md`.

## Getting Started

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/your-username/nextcloud-vpn-tunnel.git
    ```
2.  **Configure the client and server:**
    -   Copy `config/config_example.json` to `config/config.json`.
    -   Edit `config/config.json` with your Nextcloud URL, credentials, sync folder paths, and a secret encryption key.

## Disclaimer

*   **High Latency:** Due to the nature of file synchronization, this tunnel will have very high latency. It is not suitable for real-time applications like gaming or VoIP.
*   **Experimental:** This is an experimental project. Use it at your own risk. Security and reliability are not guaranteed.