# UCAS_CN_LAB - HTTP Server Experiment (English)

This document describes the experiment under `UCAS_CN_LAB` in English. It includes the experimental requirements, the main functions/modules to understand, and what needs to be implemented. It excludes any “completed report” style sections such as results/conclusions/summaries.

## Experimental Requirements

- Use C to implement the simplest HTTP server.
- Support HTTP on port `80`.
- Support HTTPS on port `443`.
- Run the HTTP and HTTPS services concurrently, using two separate execution units (e.g., two threads or two processes), with each one listening on its corresponding port.
- Support only the `GET` method.
- Parse the incoming request message and return an appropriate HTTP response and the requested content.
- Use OpenSSL to provide HTTPS functionality.

## Functions (Modules) to Understand

- `main` (start two concurrent servers and keep them running)
- `http_server` (HTTP listener loop)
- `https_server` (HTTPS listener loop)
- `Socket`, `Bind`, `Listen`, `Accept`
- `Recv`/`Send` for plain HTTP
- `SSL_Read`/`SSL_Write` for HTTPS
- `Close` for closing file descriptors/sockets
- `load_SSL` to initialize and load the server-side SSL/TLS context (certificate and private key)
- `parse_https_request` (extract `method`, `url`, `version`, and optionally `Range` header information)
- `extract_filename_from_url` to derive a local filename from the request URL
- `get_filetype` to map filename extensions to `Content-Type`
- `http_handle_client` to handle one HTTP client request (for `GET`)
- `https_handle_client` to handle one HTTPS client request (for `GET`)
- Serve static files by reading the requested file from disk and writing the response body
- Optionally support byte-range requests via `Range` for `Partial Content` (`206`) and invalid ranges for `416`

## What Needs to Be Implemented

### 1) Server startup and concurrency

- Create an HTTP server.
- Create a TCP socket for HTTP.
- Bind the HTTP socket to port `80`.
- Listen for incoming HTTP connections.
- Accept connections and dispatch each `GET` request to request-handling logic.
- Create an HTTPS server.
- Create a TCP socket for HTTPS.
- Bind the HTTPS socket to port `443`.
- Load an SSL context via OpenSSL.
- Accept HTTPS connections, perform TLS handshake, and dispatch each `GET` request to request-handling logic.
- Run HTTP and HTTPS servers concurrently (two independent execution units).

### 2) HTTP request handling (GET only)

- Read the request from the client.
- Parse the request line and extract `method`, `url`, `version`.
- Determine the requested resource from `url`.
- Determine the `Content-Type` from the resource extension.
- Generate the correct HTTP response headers.
- Read the resource content from disk and send it as the response body.
- For unsupported methods, respond with an appropriate error status (as defined by your project’s expectation).

### 3) HTTPS request handling (GET only)

- Read the HTTPS request from the TLS connection.
- Parse `method`, `url`, `version`, and optionally `Range` header.
- Resolve the requested file path from `url`.
- Determine `Content-Type` from the filename extension.
- Reply with `200 OK` and send the full file content.
- Reply with `206 Partial Content`, include `Content-Range`, and send only the requested byte range.
- Reply with `416 Range Not Satisfiable` (and do not send the file body).
- Send the response using `SSL_Write` and close/free TLS objects appropriately.

### 4) OpenSSL integration
- Create an SSL context using a server-side TLS method.
- Load the server certificate and private key files.
- Verify the private key matches the certificate.
- Ensure TLS handshake is performed for each HTTPS client connection.

### 5) Response correctness considerations
- Use consistent `Content-Type` for static file responses.
- Correctly fill `Content-Length` (and `Content-Range` when using `Range`).
- Ensure that the server returns a complete response (headers + body) for each handled `GET`.

