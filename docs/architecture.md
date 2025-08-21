# Architecture

## 1. High-level idea

You want to use Nextcloud file sync as the transport mechanism for a VPN-like tunnel. In other words:

-   **Nextcloud Server** acts as the “VPN server” and a file sync server.
-   **Client program** runs on your local device (office PC, laptop, etc.) and interacts with Nextcloud via its sync API or filesystem sync.
-   Traffic from the client is encoded/encrypted into files (or chunks) in a Nextcloud-synced folder.
-   **Nextcloud Server** decodes these files and reconstructs the traffic, forwarding it to the actual VPN server running on the same machine.
-   Traffic from the server back to the client is similarly encoded into files and synced back.

Effectively, Nextcloud sync + encryption + file encoding = encrypted “store-and-forward” tunnel.

## 2. Components

| Component            | Role                                                                                                                   |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **Nextcloud server** | Exposes a folder for sync; receives files from client and delivers files back.                                         |
| **VPN service on server** | Runs traditional VPN software (OpenVPN, WireGuard, or custom TCP/UDP endpoint).                                        |
| **Client program**   | Intercepts local traffic, encodes/encrypts it into files for Nextcloud, decodes incoming files, injects traffic to local VPN interface. |
| **Cloudflare (optional)** | Provides a public endpoint for Nextcloud via HTTPS (helps with NAT traversal, encryption, DDOS protection).          |

## 3. Workflow

### Client → Server

1.  User traffic arrives at local VPN client interface.
2.  **Client program**:
    -   Captures packets from the VPN interface.
    -   Encrypts and encodes them (e.g., AES + Base64).
    -   Writes them to files in the Nextcloud folder.
3.  Nextcloud client syncs the file to the server over HTTPS (or WebDAV).
4.  **Server program** detects new files, decodes/decrypts, and injects traffic into the VPN server interface.

### Server → Client

1.  VPN server sends packets toward the client’s virtual VPN interface.
2.  **Server program**:
    -   Captures these packets.
    -   Encrypts/encodes into files in the synced folder.
3.  Nextcloud server syncs files to the client.
4.  **Client program** detects new files, decodes/decrypts, and injects packets into the local VPN interface.

## 4. Characteristics

| Feature      | Expected behavior                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------------------------- |
| **Latency**  | High — depends on Nextcloud sync speed and chunking. Not suitable for real-time gaming or VoIP, but fine for file transfer, browsing, or async tasks. |
| **Throughput** | Limited by HTTPS sync, file chunk size, server/network bandwidth. Can be improved by splitting traffic across multiple files or directories. |
| **Stealth**  | Appears as normal Nextcloud sync traffic over HTTPS, encrypted by both your protocol and Nextcloud/TLS.         |
| **Security** | Dual encryption: VPN payload + Nextcloud TLS. End-to-end confidentiality preserved.                           |
| **Reliability** | Files must sync fully before decoding; conflicts or partial syncs may require handling.                       |

## 5. Implementation considerations

-   **Chunking & encoding**
    -   Use fixed-size chunks (e.g., 4–16 KB) to avoid extremely large files.
    -   Encode as binary or Base64 to store in plain files.
-   **File naming**
    -   Can be sequential numbers, timestamps, or GUIDs to avoid conflicts.
    -   Avoid overwriting files before they are fully synced.
-   **File cleanup**
    -   Remove processed files to prevent OneDrive/Nextcloud from storing unnecessary history.
-   **Concurrency**
    -   Multiple small files allow better throughput, but increase sync metadata overhead.
    -   Single large file is simpler but higher latency.
-   **Client/server programs**
    -   Must handle encryption/decryption, file detection, and packet injection/extraction.
    -   Can use libraries like `libpcap`/`tun`/`tap` interfaces on client and server.
-   **Public exposure**
    -   Cloudflare provides HTTPS endpoint to server.
    -   Ensure Cloudflare handles only Nextcloud traffic, and server is secured.
