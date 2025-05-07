# JS-P2P-Menu

This project is a single-HTML file implementation of a peer-to-peer (P2P) web chat application. It utilizes WebRTC via the PeerJS library to enable users to connect directly and exchange messages without a central server relaying the chat content (beyond initial signaling for connection setup).

## Features

*   **Peer-to-Peer Communication:** Messages are sent directly between connected users using WebRTC DataChannels.
*   **Host/Client Model:** One user acts as a "host" by generating and sharing their unique PeerJS ID. Other users act as "clients" by using this ID to connect to the host.
*   **Dynamic Display Names:** Users can set and update their display names. Name changes are propagated to other connected peers, and past messages in the chat log are updated to reflect the new name.
*   **Chat History:** New clients connecting to a host receive recent chat history.
*   **Dark/Light Theme:** A toggleable UI theme (dark or light) with the user's preference stored in `localStorage`.
*   **Collapsible UI:** The chat module can be minimized to a small icon.
*   **UTF-8 Message Encoding:** Messages, including emojis and international characters, are explicitly encoded/decoded as UTF-8 to ensure correct transmission.
*   **Local Storage Management:**
    *   Stores user preferences (theme, display name, last PeerJS ID).
    *   Provides an option to clear all application-specific data from `localStorage`.
*   **Cross-Browser Compatibility:** Works in modern web browsers that support WebRTC.

## How It Works

1.  **Signaling with PeerJS:**
    *   PeerJS simplifies WebRTC by managing the signaling process. It uses a signaling server to help peers discover each other and exchange the necessary metadata (like SDP offers/answers and ICE candidates) to establish a direct P2P connection.
    *   This application uses a public PeerJS signaling server and public STUN servers (for NAT traversal).
2.  **Peer Identity (PeerJS ID):**
    *   Each user's browser instance is assigned a unique PeerJS ID when it connects to the PeerJS signaling server. This ID can be randomly generated or, if previously stored, an attempt is made to reuse it.
3.  **Establishing Connections:**
    *   **Host Role:** One user clicks "Become Host." Their browser initializes a PeerJS object and waits for incoming connections. Their PeerJS ID is displayed, which they can share with others.
    *   **Client Role:** Other users paste the Host's ID into the designated input field and click "Connect." Their browser initializes its own PeerJS object and attempts to establish a `DataConnection` to the host using the provided ID.
    *   Client display names are sent as metadata during the connection request.
4.  **Direct Data Channels (`RTCDataChannel`):**
    *   Once the signaling process is complete, a direct WebRTC `RTCDataChannel` is established between each client and the host.
5.  **Message Flow:**
    *   **Client to Host:** When a client sends a message, it's sent directly to the host over their data channel.
    *   **Host to Clients:** When the host sends a message, or receives a message from one client, the host relays/broadcasts this message to all other connected clients.
    *   **Name Updates:** When a user changes their display name, a special `'name-update'` message is sent. The host relays this to other clients. Peers receiving this update will refresh their local record of the sender's name and update any past messages from that sender in their chat log.
6.  **Data Serialization and Encoding:**
    *   All message objects (chat messages, name updates, control messages) are first converted to a JSON string.
    *   This JSON string is then encoded into a UTF-8 byte array (`Uint8Array`) using `TextEncoder`.
    *   PeerJS is configured to use `'binary'` serialization for data channels, meaning it transmits this raw `Uint8Array`.
    *   On the receiving end, the `ArrayBuffer` (containing the `Uint8Array`) is decoded back to a JSON string using `TextDecoder`, and then parsed back into a JavaScript object. This ensures robust handling of all characters, including emojis.

## Technology Stack

*   **HTML5:** Provides the semantic structure for the chat application.
*   **CSS3:** Styles the user interface, including dynamic theming with CSS variables.
*   **JavaScript (ES6+):** Implements all core application logic, UI interactions, PeerJS integration, and data handling.
*   **PeerJS (v1.4.7):** A JavaScript library that abstracts the complexities of WebRTC, making it easier to establish peer-to-peer data and media connections.
    *   *Website:* [peerjs.com](https://peerjs.com/)
    *   *GitHub:* [github.com/peers/peerjs](https://github.com/peers/peerjs)
    *   *License:* MIT License
*   **WebRTC (Web Real-Time Communication):** The underlying browser API that enables direct P2P communication of audio, video, and arbitrary data between browsers.

## Usage Instructions

1.  **Open the HTML File:**
    *   Open the `index.html` (or the appropriately named HTML file) in at least two different browser windows or tabs.
    *   For testing across different devices, ensure they are on the same network (or that NAT traversal is successful if on different networks).
2.  **Set Display Name (Recommended):**
    *   In each browser window, type a desired display name into the "Your Name" input field.
    *   Click "Set Name." This name will be used to identify you in the chat and will be shared with other participants.
3.  **One User Becomes Host:**
    *   In one browser window, click the "Become Host" button.
    *   The status message will update, and the Host's full PeerJS ID will be displayed (usually below the status, and the "Copy Host ID" button will appear).
4.  **Share Host ID:**
    *   The host should copy this ID (using the "Copy Host ID" button or manually).
    *   Share this ID with the other user(s) who wish to connect (e.g., via a separate messaging app, email, or verbally).
5.  **Other Users Connect as Clients:**
    *   In the other browser window(s), paste the Host's ID into the "Host Connection ID" input field.
    *   Click the "Connect" button.
6.  **Chatting:**
    *   Once a connection is established (indicated by system messages and status updates), users can type messages into the input field at the bottom and press Enter or click "Send."
    *   Messages will appear in the chat log, identified by the sender's display name (or their short PeerJS ID if no name is set).
7.  **Interface Controls:**
    *   **Theme Toggle (☀️/🌙):** Switch between light and dark user interface themes.
    *   **Clear Stored Data (🗑️):** Clears application-specific data (theme, display name, PeerID) from the browser's `localStorage` and reloads the page. A confirmation modal will appear.
    *   **Collapse/Expand Chat (💬/✖️):** Minimizes the chat window to an icon or expands it.

## Single-File Implementation

This application is intentionally designed as a single HTML file. This approach aims to:
*   Provide a clear, self-contained example of P2P chat functionality.
*   Make it easy for others to understand, modify, and adapt the code for their own needs.
*   Serve as a learning tool for WebRTC and PeerJS concepts.

## Potential Enhancements & Future Development

*   **Modularization:** Extracting the core chat logic into a reusable JavaScript module or library.
*   **File Sharing:** Implementing P2P file transfer capabilities.
*   **Direct Client-to-Client Messaging (Mesh/Partial Mesh):** Allowing clients to message each other directly in a multi-party chat, rather than all messages being relayed by the host. This would require more complex connection management.
*   **Audio/Video Chat:** Integrating WebRTC's media stream capabilities.
*   **Improved Presence Management:** More sophisticated tracking of online/offline status of users.
*   **Persistent Chat Rooms/IDs:** A mechanism for creating and rejoining chat sessions without relying on transient PeerJS IDs.
*   **Enhanced Security:** Implementing end-to-end encryption verification or user authentication mechanisms (though WebRTC data channels are typically encrypted with DTLS by default).

## Important Considerations for Production

*   **Signaling Server:** The public PeerJS signaling server is convenient for development and small-scale use. For production applications, **deploying your own instance of PeerServer** is highly recommended for reliability, scalability, and control.
*   **STUN/TURN Servers:** While public STUN servers (like `stun:stun.l.google.com:19302`) assist with NAT traversal, they may not be sufficient for all network configurations. A **TURN server** (Traversal Using Relays around NAT) may be necessary to relay traffic if a direct P2P connection cannot be established. Operating your own TURN server incurs costs.
*   **Scalability:** The current host-client relay model has limitations on the number of participants the host's browser can efficiently manage. For larger chats, different architectures (e.g., SFU/MCU for media, or a more distributed data relay) would be needed.
*   **Error Handling & Resilience:** The application includes basic error handling. Production-grade systems require more robust strategies for connection drops, retries, and user feedback.

## License

This project is licensed under the MIT License.


MIT License

Copyright (c) 2025 BadFoxAI

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
