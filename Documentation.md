# PermastoreIt 1.2.0 Documentation

![alt text](PermastoreIt.jpg)

## Table of Contents

1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Installation](#installation)
   - [Prerequisites](#prerequisites)
   - [Standard Installation](#standard-installation)
   - [Docker Installation](#docker-installation)
4. [Configuration](#configuration)
5. [API Reference](#api-reference)
   - [File Operations](#file-operations)
   - [System Status](#system-status)
   - [Zero-Knowledge Proofs](#zero-knowledge-proofs)
6. [SDK Reference](#sdk-reference)
   - [Installation](#sdk-installation)
   - [Basic Usage](#basic-usage)
   - [Client Methods](#client-methods)
   - [Error Handling](#error-handling)
   - [Examples](#examples)
7. [Core Components](#core-components)
   - [Server](#server)
   - [P2P Node](#p2p-node)
   - [Blockchain](#blockchain)
   - [Network](#network)
   - [AI Optimizer](#ai-optimizer)
   - [ZKP Verifier](#zkp-verifier)
8. [Storage Structure](#storage-structure)
9. [Security Features](#security-features)
10. [Monitoring and Maintenance](#monitoring-and-maintenance)
11. [Troubleshooting](#troubleshooting)
12. [Development Guide](#development-guide)
13. [Best Practices](#best-practices)
14. [Changelog](#changelog)
15. [License](#license)

## Introduction

PermastoreIt 1.2.0 is a decentralized file storage system that uses blockchain technology and peer-to-peer networking to provide resilient, tamper-proof file storage and distribution. It incorporates advanced features like AI-based file deduplication, zero-knowledge proofs for verification, and content-addressed storage.

Key features:

- **Content-addressed storage**: Files are identified by their cryptographic hash
- **Blockchain ledger**: Immutable record of all file transactions
- **Kademlia DHT**: Distributed Hash Table for efficient P2P file discovery and retrieval
- **AI-powered deduplication**: Automatically detects similar files
- **Zero-knowledge proofs**: Verify file existence without revealing the file
- **FastAPI-based server**: Modern, high-performance REST API
- **Python SDK**: Programmatic access to all functionality

## Architecture Overview

PermastoreIt 1.2.0 follows a modular architecture with six main components:

1. **Server**: Provides the HTTP API for interacting with the system
2. **P2P Node**: Orchestrates file storage and retrieval operations
3. **Blockchain**: Maintains an immutable ledger of file transactions
4. **Network**: Handles peer-to-peer communication using Kademlia DHT
5. **AI Optimizer**: Provides intelligent file deduplication and tagging
6. **ZKP Verifier**: Generates and verifies zero-knowledge proofs

```mermaid
flowchart TD
    %% UPLOAD FLOW
    subgraph "Upload Flow"
        A[Client] -->|Upload Request + File| B[Server]
        B -->|Receives File| C[P2P Node]
        C -->|Analyze File| D[AI Optimizer]
        D -->|File Analysis| E{Unique?}
    
        E -->|Yes| F[Local Storage]
        E -->|Yes| G[Blockchain]
        F -->|Save File| H[File Saved]
        G -->|Add Transaction| I[TX Added]
    
        E -->|Yes| J[Network/DHT]
        J -->|Announce File/Hash| K[Other Peers]
    
        E -->|No| L[Response: Deduplicated]
    
        %% Optional ZKP
        C -.->|Optional| M[ZKP Generator]
        M -.->|Generate Proof| N[ZKP Verifier]
        N -.->|Verified| O[Proof]
    end
  
    %% RETRIEVAL FLOW
    subgraph "Retrieval Flow"
        AA[Client] -->|Download Request + Hash| BB[Server]
        BB -->|Get File Request| CC[P2P Node]
        CC -->|Check| DD{Found Locally?}
    
        DD -->|Yes| EE[Send File]
        EE -->|Return| FF[Client Receives File]
    
        DD -->|No| GG[Network/DHT]
        GG -->|Look up Hash in DHT| HH[Other Peers]
        HH -->|Peer Responds| II[P2P Node]
        II -->|Forward File| FF
    
        HH -->|Not Found| JJ[Response: Not Found]
        JJ -->|Return| KK[Client Notified]
    end
  
    %% Styling
    classDef server fill:#f96,stroke:#333,stroke-width:2px
    classDef p2pnode fill:#9cf,stroke:#333,stroke-width:2px
    classDef storage fill:#9f9,stroke:#333,stroke-width:2px
    classDef blockchain fill:#c9f,stroke:#333,stroke-width:2px
    classDef client fill:#fc9,stroke:#333,stroke-width:2px
    classDef network fill:#ccc,stroke:#333,stroke-width:2px
    classDef decision fill:#fff,stroke:#333,stroke-width:2px
  
    class A,AA,FF,KK client
    class B,BB server
    class C,CC,II p2pnode
    class F,H storage
    class G,I blockchain
    class J,GG,HH,K network
    class E,DD decision
```

The data flow in the system:

1. A client uploads a file to a PermastoreIt node via the HTTP API or SDK
2. The file is analyzed for duplicates using the AI Optimizer
3. If unique, the file is stored locally and its hash recorded in the blockchain
4. The file hash is announced to the Kademlia DHT network
5. A zero-knowledge proof can be generated for verification
6. Files can be retrieved from any node using their hash, with DHT lookup if not found locally

## Configuration

PermastoreIt is configured via the `config.json` file. Here are the available options:

```json
{
  "upload_dir": "uploads",         // Directory where files are stored
  "max_file_size": 104857600,      // Maximum file size in bytes (100MB)
  "host": "0.0.0.0",               // Host to bind the server to
  "port": 5000,                    // Port to listen on
  "debug": false,                  // Enable debug mode
  "allowed_file_types": [          // Allowed MIME types
    "application/pdf",
    "image/jpeg",
    "image/png",
    "text/plain",
    "application/json",
    "application/octet-stream"
  ],
  "blockchain": {
    "storage_file": "data/blockchain.json",  // Blockchain storage file
    "min_transactions_per_block": 1     // Min transactions per block
  },
  "network": {
    "peer_file": "data/peers.txt",     // File storing peer information
    "retry_limit": 3,              // Network retry attempts
    "request_timeout": 30,         // Request timeout in seconds
    "sync_interval": 3600          // Sync interval in seconds
  },
  "logging": {
    "level": "INFO",               // Logging level
    "file": "logs/permastore.log", // Log file
    "max_size": 10485760,          // Max log size (10MB)
    "backup_count": 5,             // Number of log backups
    "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s" // Log format
  },
  "ai": {
    "min_similarity": 0.85         // Minimum similarity threshold
  },
  "zkp": {
    "enabled": true                // Enable zero-knowledge proofs
  },
  "cleanup_interval": 86400        // Interval for cleanup operations (24 hours)
}
```

## API Reference

PermastoreIt 2.0 provides a RESTful API (built with FastAPI) for interacting with the system. All API endpoints return JSON responses unless otherwise specified.

### File Operations

#### Upload a File

```
POST /upload
```

Upload a file to the PermastoreIt network.

**Request:**

- Method: POST
- Content-Type: multipart/form-data
- Body: Form data with a "file" field containing the file to upload
- Rate limit: 10 requests per minute

**Response:**

```json
{
  "status": "success",
  "hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "size": 1024,
  "zkp_available": true,
  "message": "File uploaded and stored successfully"
}
```

**Status Codes:**

- 201: File uploaded and stored successfully
- 200: File already exists or was deduplicated
- 400: Invalid request (missing file, invalid type)
- 413: File too large
- 500: Internal server error
- 503: Node not initialized

#### Download a File

```
GET /download/{file_hash}
```

Download a file from the PermastoreIt network by its hash.

**Request:**

- Method: GET
- URL Parameters: file_hash - The hash of the file to download
- Rate limit: 60 requests per minute

**Response:**

- The file content with appropriate Content-Type and Content-Disposition headers

**Status Codes:**

- 200: File found and returned
- 404: File not found locally or via DHT
- 500: Internal server error
- 503: Node not initialized

#### Get File Information

```
GET /file-info/{file_hash}
```

Get metadata about a file stored in the PermastoreIt network.

**Request:**

- Method: GET
- URL Parameters: file_hash - The hash of the file
- Rate limit: 60 requests per minute

**Response:**

```json
{
  "hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "filename": "example.txt",
  "content_type": "text/plain",
  "size": 1024,
  "timestamp": 1648500000.0,
  "tags": ["text", "document", "plain"]
}
```

**Status Codes:**

- 200: File information retrieved successfully
- 404: File not found
- 500: Internal server error
- 503: Node not initialized

#### List All Files

```
GET /files
```

List metadata for all files stored in the node.

**Request:**

- Method: GET
- Query Parameters: limit (optional) - Maximum number of files to return
- Rate limit: 30 requests per minute

**Response:**

```json
[
  {
    "hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "filename": "example.txt",
    "content_type": "text/plain",
    "size": 1024,
    "timestamp": 1648500000.0,
    "tags": ["text", "document", "plain"]
  },
  {
    "hash": "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824",
    "filename": "another.pdf",
    "content_type": "application/pdf",
    "size": 5242880,
    "timestamp": 1648499000.0,
    "tags": ["pdf", "document", "report"]
  }
]
```

**Status Codes:**

- 200: Files listed successfully
- 500: Internal server error
- 503: Node not initialized

#### Search Files

```
GET /search
```

Search for files based on filename or tags.

**Request:**

- Method: GET
- Query Parameters: 
  - query - The search query (min length: 2)
  - limit - Maximum number of results (default: 10, max: 50)
- Rate limit: 30 requests per minute

**Response:**

```json
[
  {
    "hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "filename": "example.txt",
    "content_type": "text/plain", 
    "size": 1024,
    "similarity": 0.95
  },
  {
    "hash": "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824",
    "filename": "examples.json",
    "content_type": "application/json",
    "size": 2048,
    "similarity": 0.87
  }
]
```

**Status Codes:**

- 200: Search completed successfully
- 500: Internal server error
- 503: Node not initialized

### System Status

#### Get Status

```
GET /status
```

Get basic status information about the node.

**Request:**

- Method: GET

**Response:**

```json
{
  "status": "operational",
  "node_id": "7a28b6e9a4a1",
  "files_stored": 23,
  "blockchain_length": 15,
  "peers_connected": 5
}
```

**Status Codes:**

- 200: Status retrieved successfully
- 500: Internal server error
- 503: Node not initialized

#### Health Check

```
GET /health
```

Perform a comprehensive health check of all node components.

**Request:**

- Method: GET

**Response:**

```json
{
  "status": "healthy",
  "components": {
    "blockchain": true,
    "network": true,
    "ai": true,
    "storage": true,
    "disk_space": true,
    "dht": true
  },
  "node_id": "7a28b6e9a4a1",
  "files_stored": 23,
  "blockchain_length": 15,
  "peers_connected": 5
}
```

**Status Codes:**

- 200: Node is healthy
- 503: Node has degraded functionality
- 500: Health check failed

### Zero-Knowledge Proofs

#### Generate Proof

```
GET /zk-proof/{file_hash}
```

Generate a zero-knowledge proof for a file.

**Request:**

- Method: GET
- URL Parameters: file_hash - The hash of the file
- Rate limit: 30 requests per minute

**Response:**

```json
{
  "proof": "7a28b6e9a4a19b5b2dac427fca7b6478ec1a4e2dfcd048210ee43c7a7f1d5b52",
  "challenge": "89a6b3c45e2d1f7d",
  "algorithm": "SHA256-HKDF",
  "status": "success"
}
```

**Status Codes:**

- 200: Proof generated successfully
- 404: File not found
- 501: ZKP generation not implemented or disabled
- 500: Internal server error
- 503: Node not initialized

## SDK Reference

The PermastoreIt Python SDK provides a convenient way to interact with the PermastoreIt API programmatically. It handles HTTP communication, file operations, and error management.

### SDK Installation

To install the SDK:

```bash
# From PyPI (Recommended)
pip install permastoreit-sdk

# From source
git clone https://github.com/permastoreit/sdk-python.git
cd sdk-python
pip install -e .
```

### Basic Usage

```python
from permastoreit_sdk import PermastoreItClient

# Create a client instance
client = PermastoreItClient(base_url="http://localhost:5000")

# Check the node status
status = client.get_status()
print(f"Node status: {status}")

# Upload a file
upload_result = client.upload("/path/to/file.txt")
file_hash = upload_result["hash"]
print(f"File uploaded successfully, hash: {file_hash}")

# Download a file
download_path = client.download(file_hash, save_dir="downloads")
print(f"File downloaded to: {download_path}")
```

### Client Methods

The `PermastoreItClient` class provides the following methods:

#### Initialization

```python
client = PermastoreItClient(base_url="http://localhost:5000", timeout=60)
```

- `base_url`: The base URL of the PermastoreIt node API (default: "http://localhost:5000")
- `timeout`: Default timeout in seconds for API requests (default: 60)

#### Get Root Message

```python
response = client.get_root_message()
```

Returns the welcome message from the root endpoint.

#### Get Status

```python
status = client.get_status()
```

Returns the operational status of the node with details about files stored, blockchain length, etc.

#### Get Health

```python
health = client.get_health()
```

Returns a comprehensive health check report of all node components.

#### Upload

```python
result = client.upload(file_path)
```

Uploads a file from the given local path to the node.

- `file_path`: The local path to the file to upload.

Returns a dictionary containing the upload result (status, hash, size, etc.).

#### Download

```python
saved_path = client.download(file_hash, save_dir, save_filename=None)
```

Downloads a file by its hash and saves it to a specified directory.

- `file_hash`: The SHA-256 hash of the file to download.
- `save_dir`: The directory where the file should be saved. Created if it doesn't exist.
- `save_filename`: Optional. The name to save the file as. If None, uses the file hash.

Returns the full path to the downloaded file.

#### List Files

```python
files = client.list_files(limit=None)
```

Retrieves metadata for stored files, optionally limited to a specified number.

- `limit`: Optional maximum number of recent files to return.

Returns a list of file metadata dictionaries.

#### Get File Info

```python
info = client.get_file_info(file_hash)
```

Gets metadata for a specific file hash from the blockchain record.

- `file_hash`: The SHA-256 hash of the file.

Returns a dictionary containing the file's metadata.

#### Search

```python
results = client.search(query, limit=10)
```

Searches for files by query (matches filenames and tags).

- `query`: The search term.
- `limit`: The maximum number of results to return (default 10).

Returns a list of search result dictionaries, sorted by relevance.

#### Get ZK Proof

```python
proof = client.get_zk_proof(file_hash)
```

Gets the Zero-Knowledge Proof for a file hash.

- `file_hash`: The SHA-256 hash of the file.

Returns a dictionary containing the ZKP details (proof, challenge, algorithm).

### Error Handling

The SDK defines custom exceptions to handle different error scenarios:

#### PermastoreItError

Base exception for all SDK errors.

#### APIError

Raised for non-2xx API responses.

```python
try:
    client.get_status()
except APIError as e:
    print(f"API error {e.status_code}: {e.detail}")
```

#### NetworkError

Raised for connection, timeout, or other request errors.

```python
try:
    client.upload("example.txt")
except NetworkError as e:
    print(f"Network error: {e}")
```

#### FileNotFoundErrorOnServer

Raised specifically for 404 errors when expecting a file/resource.

```python
try:
    client.download("nonexistent_hash", "downloads")
except FileNotFoundErrorOnServer as e:
    print(f"File not found: {e.resource_id}")
```

#### ZKPDisabledError

Raised when ZKP functionality is requested but disabled on the server.

```python
try:
    client.get_zk_proof(file_hash)
except ZKPDisabledError:
    print("Zero-Knowledge Proofs are disabled on this server")
```

### Examples

#### Complete File Upload and Download

```python
from permastoreit_sdk import PermastoreItClient, FileNotFoundErrorOnServer

client = PermastoreItClient(base_url="http://localhost:5000")

try:
    # Upload a file
    result = client.upload("my_document.pdf")
    file_hash = result["hash"]
    print(f"File uploaded successfully with hash: {file_hash}")
    
    # Get file information
    info = client.get_file_info(file_hash)
    print(f"File name: {info['filename']}")
    print(f"File size: {info['size']} bytes")
    print(f"Content type: {info['content_type']}")
    
    # Download the file
    download_path = client.download(file_hash, save_dir="downloads", save_filename="downloaded_document.pdf")
    print(f"File downloaded to: {download_path}")

except FileNotFoundErrorOnServer as e:
    print(f"File not found on server: {e}")
except Exception as e:
    print(f"Error: {e}")
```

#### Search and List Files

```python
from permastoreit_sdk import PermastoreItClient

client = PermastoreItClient(base_url="http://localhost:5000")

# List recent files (limit 5)
recent_files = client.list_files(limit=5)
print("Recent files:")
for file in recent_files:
    print(f" - {file['filename']} ({file['hash']})")

# Search for PDF files
pdf_files = client.search("pdf", limit=10)
print("\nPDF files:")
for file in pdf_files:
    print(f" - {file['filename']} (Similarity: {file['similarity']})")
```

#### Zero-Knowledge Proofs

```python
from permastoreit_sdk import PermastoreItClient, ZKPDisabledError

client = PermastoreItClient(base_url="http://localhost:5000")

try:
    # Upload a file first
    result = client.upload("confidential_document.txt")
    file_hash = result["hash"]
    
    # Get zero-knowledge proof
    proof = client.get_zk_proof(file_hash)
    print(f"ZKP proof generated: {proof['proof']}")
    print(f"Challenge: {proof['challenge']}")
    print(f"Algorithm: {proof['algorithm']}")
    
except ZKPDisabledError:
    print("Zero-Knowledge Proofs are disabled on this server")
except Exception as e:
    print(f"Error: {e}")
```

## Core Components

### Server

The server component (`server.py`) is the entry point for the application and provides the HTTP API for interacting with the PermastoreIt network. It's built using FastAPI, a modern, high-performance web framework for Python.

Key responsibilities:

- Expose RESTful API endpoints
- Handle request validation and rate limiting
- Route requests to the appropriate components
- Format and send responses
- Handle errors and exceptions
- Manage the Kademlia DHT server lifecycle

The server uses asyncio and supports asynchronous operations for better performance, particularly for network and I/O operations.

### P2P Node

The P2P Node component (`p2p_node.py`) coordinates interactions between all other components. It manages file storage, retrieval, and distribution across the network.

Key responsibilities:

- Store and retrieve files from the local filesystem
- Calculate file hashes
- Record file transactions in the blockchain
- Coordinate with the AI Optimizer for file deduplication
- Generate zero-knowledge proofs via the ZKP Verifier
- Interface with the Kademlia DHT for distributed file discovery

### Blockchain

The blockchain component (`blockchain.py`) maintains a tamper-proof ledger of all file transactions. It implements a simple blockchain structure where each block contains a list of transactions, a timestamp, and a reference to the previous block.

Key responsibilities:

- Create new blocks in the blockchain
- Add file transactions to blocks
- Calculate cryptographic block hashes
- Persist blockchain data to disk
- Load blockchain data from disk

### Network

The network component (`network.py`) handles communication between nodes in the PermastoreIt network. It implements a Kademlia Distributed Hash Table (DHT) for efficient peer discovery and file lookup.

Key responsibilities:

- Start and manage the Kademlia DHT server
- Announce file hashes to the DHT
- Look up files in the DHT
- Bootstrap the DHT network
- Maintain routing tables for efficient peer communication

### AI Optimizer

The AI Optimizer component (`ai_optimizer.py`) provides intelligent file analysis, deduplication, and tagging using machine learning models.

Key responsibilities:

- Calculate similarity between files
- Detect duplicate or near-duplicate files
- Generate semantic tags for files
- Optimize storage by preventing redundancy

### ZKP Verifier

The ZKP (Zero-Knowledge Proof) Verifier component (`zkp.py`) provides cryptographic proofs that allow verification of file existence without revealing the file content.

Key responsibilities:

- Generate cryptographic proofs for files
- Verify proofs provided by peers
- Enhance security and privacy of the system

## Storage Structure

PermastoreIt uses a content-addressed storage system, where files are identified by their cryptographic hash. The directory structure is simple but effective:

```
uploads/
├── e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
├── 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
└── ...

data/
├── blockchain.json
└── peers.txt

logs/
└── permastore.log
```

Each file is stored with its SHA-256 hash as the filename, making it easy to locate and retrieve files by their hash. This approach also provides implicit deduplication at the file level, as identical files will have identical hashes.

The blockchain data is stored in the `data/blockchain.json` file, which contains the complete blockchain including all transactions. The list of peers is stored in the `data/peers.txt` file, and logs are written to `logs/permastore.log`.

## Security Features

PermastoreIt 1.2.0 includes several security features to protect the integrity and availability of stored data:

1. **Content Integrity**: Files are identified by their cryptographic hash, making it easy to verify that files haven't been tampered with.
2. **Blockchain Ledger**: All file transactions are recorded in an immutable blockchain, providing a tamper-proof audit trail.
3. **Zero-Knowledge Proofs**: The ZKP system allows verification of file existence without revealing file contents or metadata.
4. **Hash Verification**: When downloading files from peers, the hash is verified to ensure the file hasn't been tampered with.
5. **Duplicate Detection**: The AI Optimizer can detect not just identical files, but also near-duplicates, preventing storage of slightly modified versions of the same content.
6. **File Type Validation**: The system validates file types and only allows specified MIME types to be uploaded.
7. **File Size Limits**: Maximum file size is enforced to prevent denial-of-service attacks.
8. **Rate Limiting**: The API implements rate limiting to prevent abuse (using the SlowAPI library).
9. **Trusted Host Middleware**: The server uses FastAPI's TrustedHostMiddleware to prevent host header attacks.
10. **CORS Protection**: Cross-Origin Resource Sharing is configured to allow only specific origins.

For production deployments, consider implementing these additional security measures:

1. **HTTPS**: Use a reverse proxy (like Nginx) with proper TLS/SSL configuration.
2. **Authentication**: Implement an access control system.
3. **Network Segmentation**: Separate public-facing components from internal storage.
4. **Regular Backups**: Maintain regular backups of blockchain data.

## Monitoring and Maintenance

### Logging

PermastoreIt uses Python's standard logging module to record events and errors. Logs are written to the file specified in the configuration (`logs/permastore.log` by default).

Example log entry:

```
2025-03-30 12:34:56,789 - p2p_node - INFO - File stored successfully: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

The logging system includes:

- Configurable log levels (DEBUG, INFO, WARNING, ERROR, CRITICAL)
- Rotating file handler to manage log size
- Formatting options
- Component-specific loggers
- Both file and console output

### Health Checks

The `/health` endpoint provides a comprehensive way to check if the node is functioning correctly. It verifies:

1. Blockchain integrity
2. Network connectivity
3. AI optimizer functionality
4. Storage accessibility
5. Disk space availability
6. DHT server status

You can use monitoring tools like Prometheus, Grafana, or even simple scripts to periodically check this endpoint and alert if the node becomes unhealthy.

### Backup and Recovery

To backup PermastoreIt data, you should regularly backup the following files and directories:

1. `data/blockchain.json`: Contains the blockchain data
2. `data/peers.txt`: Contains the list of known peers
3. `uploads/` directory: Contains all stored files

For Docker deployments, these directories can be mounted as volumes, so they persist even if the container is removed.

## Troubleshooting

### Common Issues

#### Node Not Starting

**Symptoms**: Server fails to start, or exits immediately after starting.

**Possible causes and solutions**:

1. **Port conflict**: Another application is using the configured port.

   - Solution: Change the port in `config.json` or stop the conflicting application.
2. **Missing dependencies**: Required Python packages are not installed.

   - Solution: Run `pip install -r requirements.txt` to install all dependencies.
3. **Invalid configuration**: The `config.json` file contains invalid settings.

   - Solution: Check the logs for specific errors and correct the configuration.
4. **Permission issues**: The application doesn't have permission to write to the required directories.

   - Solution: Check and correct permissions on the `uploads`, `data`, and `logs` directories.
5. **Directory structure missing**: Required directories don't exist.

   - Solution: Create the necessary directories (`uploads`, `data`, `logs`).

#### SDK Connection Issues

**Symptoms**: The SDK fails to connect to the PermastoreIt node.

**Possible causes and solutions**:

1. **Node not running**: The PermastoreIt node is not running or is unreachable.

   - Solution: Verify the node is running and accessible.
2. **Incorrect base URL**: The base URL provided to the SDK is incorrect.

   - Solution: Check the URL (including protocol, host, and port).
3. **Network issues**: Network problems between the client and server.

   - Solution: Check network connectivity, firewalls, and proxy settings.
4. **Timeout too short**: The timeout value is too short for slow operations.

   - Solution: Increase the timeout value when creating the client.

#### DHT Not Starting

**Symptoms**: The health check shows `"dht": false` or log messages indicate DHT startup failures.

**Possible causes and solutions**:

1. **Network configuration issues**: Firewall or network settings blocking UDP traffic.

   - Solution: Check firewall settings and ensure UDP traffic is allowed.
2. **Port conflicts**: Another application is using the DHT port.

   - Solution: Configure a different DHT port if possible.
3. **Bootstrapping problems**: Cannot connect to bootstrap nodes.

   - Solution: Verify bootstrap node addresses and ensure they are reachable.
4. **Kademlia library issues**: Problems with the Kademlia implementation.

   - Solution: Check for updates to the library or consider alternative implementations.

#### Files Not Found When Downloading

**Symptoms**: Download requests return 404 Not Found, even though the file should exist in the network.

**Possible causes and solutions**:

1. **DHT issues**: The DHT is not properly announcing or finding files.

   - Solution: Check DHT status in health check, restart the node if necessary.
2. **File hash mismatch**: The requested hash doesn't match any file in the network.

   - Solution: Verify the hash and ensure it's correctly entered.
3. **No peers with the file**: None of the peers in the DHT have the requested file.

   - Solution: Upload the file to at least one node in the network.
4. **Network connectivity**: Peers cannot reach each other.

   - Solution: Check network connectivity between nodes, ensure firewalls allow traffic.

#### High Resource Usage

**Symptoms**: Node is consuming excessive CPU, memory, or disk space.

**Possible causes and solutions**:

1. **Too many files**: The node is storing a large number of files.

   - Solution: Add more storage, or distribute files across more nodes.
2. **AI optimizer overload**: The AI optimizer is consuming too many resources.

   - Solution: Adjust the `min_similarity` threshold in the configuration to reduce processing load.
3. **Blockchain size**: The blockchain has grown too large.

   - Solution: Implement a pruning mechanism for old blocks (not included in the basic version).
4. **Memory leaks**: Potential bugs causing memory leaks.

   - Solution: Update to the latest version, or examine logs for clues about problematic components.
5. **DHT traffic**: High DHT traffic causing resource consumption.

   - Solution: Adjust DHT configuration parameters or limit the scope of DHT announcements.

### Diagnostic Tools

PermastoreIt 1.2.0 includes several diagnostic endpoints to help identify and fix issues:

1. **Logs**: The first place to look for error messages and warnings.

   - View the log file specified in the configuration.
   - Set the log level to `DEBUG` in the configuration for more detailed logs.
2. **Status endpoint**: The `/status` endpoint provides information about the node's state.

   - Check the blockchain length and peer count.
   - Verify the number of files stored.
3. **Health endpoint**: The `/health` endpoint checks if all components are functioning correctly.

   - If it returns `"status": "degraded"`, check the individual component statuses.
   - Look for `false` values in the `components` section to identify specific problems.

## Development Guide

This section provides guidance for developers who want to extend or modify PermastoreIt.

### Code Structure

The codebase is organized as follows:

```
permastoreit-1.2.0/
├── server.py               # Entry point, HTTP API
├── p2p_node.py             # Core node functionality
├── blockchain.py           # Blockchain implementation
├── network.py              # Network and peer management
├── ai_optimizer.py         # AI-based file analysis
├── zkp.py                  # Zero-knowledge proof system
├── config.py               # Configuration loading and validation
├── uploads/                # File storage directory
├── data/                   # Blockchain and peer data
│   ├── blockchain.json     # Blockchain storage
│   └── peers.txt           # Peer list
├── logs/                   # Log files
│   └── permastore.log      # Main log file
├── requirements.txt        # Python dependencies
├── config.json             # Configuration file
└── Documentation.md        # Project documentation
```

### Key Asynchronous Functions

PermastoreIt 2.0 uses asyncio for improved performance. Key asynchronous functions include:

1. **Lifespan management**: Application startup and shutdown functions manage DHT lifecycle:

```python
async def lifespan(app: FastAPI):
    # Startup
    if node and hasattr(node, 'network') and hasattr(node.network, 'start'):
        node.network._dht_task = asyncio.create_task(node.network.start())
    yield
    # Shutdown
    if node and hasattr(node, 'network') and hasattr(node.network, 'stop'):
        await node.network.stop()
```

2. **File upload**: The upload endpoint is asynchronous:

```python
@app.post("/upload", response_model=UploadResponse, status_code=201)
@limiter.limit("10/minute")
async def upload_file(request: Request, file: UploadFile = File(...)):
    result = await node.store_file(file)
    return JSONResponse(content=result)
```

3. **File download**: The download function retrieves files from local storage or DHT peers:

```python
@app.get("/download/{file_hash}")
@limiter.limit("60/minute")
async def download_file(request: Request, file_hash: str):
    file_path = await node.retrieve_file(file_hash)
    if file_path and os.path.exists(file_path):
        return FileResponse(path=file_path)
```

### Adding a New API Endpoint

To add a new API endpoint:

1. Open `server.py` and define a new route using FastAPI decorators:

```python
@app.get("/new-endpoint/{parameter}", tags=["Category"])
@limiter.limit("30/minute")
async def new_endpoint(request: Request, parameter: str):
    # Validate input
    if not parameter or len(parameter) < 3:
        raise HTTPException(status_code=400, detail="Invalid parameter (min length: 3)")
  
    try:
        # Implement your endpoint logic here
        result = {"parameter": parameter, "processed": True}
        return result
    except Exception as e:
        logger.error(f"Error in new endpoint: {str(e)}", exc_info=True)
        raise HTTPException(status_code=500, detail="Internal server error during processing.")
```

2. Create appropriate response models using Pydantic:

```python
class NewEndpointResponse(BaseModel):
    parameter: str
    processed: bool
    timestamp: float = Field(default_factory=time.time)
```

3. Update the API documentation by adding appropriate docstrings.

### Extending the DHT Functionality

To enhance or extend the Kademlia DHT functionality:

1. Update the `network.py` file to add new DHT methods:

```python
async def announce_value(self, key, value):
    """
    Announce a key-value pair to the DHT.
  
    Args:
        key (str): The key to announce
        value (str): The value to associate with the key
  
    Returns:
        bool: True if announced successfully, False otherwise
    """
    if not self._dht_started:
        await self.start()
    
    try:
        await self.dht_server.set(key, value)
        return True
    except Exception as e:
        logger.error(f"Failed to announce to DHT: {e}")
        return False
```

2. Add corresponding methods to the P2P Node component:

```python
async def announce_metadata(self, file_hash, metadata):
    """
    Announce file metadata to the DHT.
  
    Args:
        file_hash (str): The file hash
        metadata (dict): The metadata to announce
  
    Returns:
        bool: True if announced successfully
    """
    metadata_json = json.dumps(metadata)
    return await self.network.announce_value(f"meta:{file_hash}", metadata_json)
```

3. Expose the functionality through the API as needed.

## Best Practices

### Performance Optimization

1. **Use asynchronous I/O**: Leverage asyncio for I/O-bound operations to improve concurrency.
2. **Optimize DHT parameters**: Fine-tune Kademlia parameters for your specific network topology.
3. **Limit AI processing**: The AI Optimizer is resource-intensive. Only use it when necessary.
4. **Implement caching**: Cache frequently accessed files and blockchain data.
5. **Use appropriate hash algorithms**: SHA-256 provides a good balance of security and performance.
6. **Monitor resource usage**: Regularly check CPU, memory, and disk usage.

### Scalability

1. **Horizontal scaling**: Add more nodes to the network to handle increased load.
2. **Load balancing**: Use a load balancer to distribute requests across multiple nodes.
3. **Optimize DHT routing**: Fine-tune DHT parameters for larger networks.
4. **Implement content replication**: Automatically replicate popular content across multiple nodes.
5. **Sharding**: Consider implementing data sharding for very large deployments.

### Security

1. **Regular security audits**: Conduct security audits of the codebase.
2. **Keep dependencies updated**: Regularly update dependencies to patch security vulnerabilities.
3. **Use secure communication**: Implement TLS/SSL for all communications.
4. **Validate all inputs**: Thoroughly validate all inputs to prevent injection attacks.
5. **Implement proper authentication**: Add user authentication for production deployments.

## Changelog

### Version 1.2.0 (April 2025)

* Implemented Kademlia DHT for improved peer-to-peer file discovery and retrieval
* Rewritten server component using FastAPI for improved performance and developer experience
* Added asynchronous API endpoints for better concurrency and throughput
* Implemented comprehensive health checks for system monitoring
* Added rate limiting to prevent API abuse
* Enhanced error handling and logging throughout the system
* Improved security with CORS and Trusted Host protection
* Added detailed API documentation with OpenAPI/Swagger integration
* Updated file retrieval process to leverage DHT for distributed lookups
* Restructured configuration with better organized directory paths

### Version 1.0.1 (Internal Update