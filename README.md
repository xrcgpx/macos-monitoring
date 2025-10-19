# macOS Monitoring with Prometheus & Grafana

This project sets up a local monitoring environment for macOS using Prometheus, Grafana, and dedicated exporters.
It visualizes both system-wide metrics and per-process metrics.

## Architecture

- **Grafana** (Docker): For visualizing metrics.
- **Prometheus** (Docker): For collecting and storing metrics.
- **Node Exporter** (Host): For collecting system-wide metrics (CPU, memory, disk, etc.).
- **Darwin Exporter** (Host): For collecting detailed macOS-specific metrics, including per-process metrics.

## Prerequisites

- [Homebrew](https://brew.sh/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)

## Setup Guide

### 1. Install Exporters on macOS Host

These exporters run directly on your Mac to collect host-level data.

#### Node Exporter (System-wide metrics)

1.  Install `node_exporter` using Homebrew:

    ```bash
    brew install node_exporter
    ```

2.  To allow connections from the Prometheus Docker container, create a configuration file that tells `node_exporter` to listen on all network interfaces.

    ```bash
    echo '--web.listen-address=:9100' | sudo tee /opt/homebrew/etc/node_exporter.args
    ```

3.  Start `node_exporter` as a background service:
    ```bash
    brew services start node_exporter
    ```

#### Darwin Exporter (macOS-specific & process metrics)

1.  Tap the custom repository:

    ```bash
    brew tap umegbewe/tap
    ```

2.  Install `darwin-exporter`:

    ```bash
    brew install darwin-exporter
    ```

3.  To get complete metrics, this exporter needs to be run with `root` privileges. Open a **dedicated terminal window** for this, as it will run in the foreground.
    `bash
    sudo /opt/homebrew/opt/darwin-exporter/bin/darwin-exporter --bind 0.0.0.0 --port 1053
    ` > **Note on `sudo brew services`**:
    > During setup, I found that running `darwin-exporter` as a root service via `sudo brew services start` was unreliable. The service would appear to start but would not listen on the specified port.
    >
    > Therefore, the most reliable method is to run the command directly in the foreground as shown above. Keep this terminal window open, as closing it will stop the exporter.

### 2. Launch Prometheus & Grafana

With the exporters running on the host, launch the Docker containers.

```bash
docker compose up -d
```

### 3. Configure Grafana

1.  Open Grafana in your browser: `http://localhost:3000`
2.  Log in with the default credentials (`admin` / `admin`) and change the password.
3.  **Add Prometheus Data Source:**

    - Navigate to **Connections** > **Data sources** > **Add new data source**.
    - Select **Prometheus**.
    - Set the **Prometheus server URL** to `http://prometheus:9090`.
    - Click **Save & test**.

4.  **Import Dashboards:**
    - Navigate to **Dashboards** > **New** > **Import**.
    - Import the following dashboards by their ID:
      - **`13978`**: A good starting point for macOS system metrics (from `node_exporter`).
      - **`24129`**: For detailed process metrics (from `darwin-exporter`).
    - For each import, be sure to select your Prometheus data source.

## Usage

- **Grafana Dashboards**: `http://localhost:3000`
- **Prometheus UI**: `http://localhost:9090`

## How to Stop

1.  Stop the Docker containers:
    ```bash
    docker compose down
    ```
2.  Stop the `node_exporter` service:
    ```bash
    brew services stop node_exporter
    ```
3.  Stop `darwin-exporter` by pressing `Ctrl+C` in the terminal window where it is running.
