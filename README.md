# 🐦 NovaNature Worker: Bioacoustics

> **The Digital Ear of the Planet.**
> Automated species identification and biodiversity monitoring from passive audio streams.

[](https://www.google.com/search?q=https://github.com/novaeco-tech/novanature-worker-bioacoustics/actions)
[](https://opensource.org/licenses/MIT)
[](https://www.google.com/search?q=https://bio.nature.novaeco.tech)

**Bioacoustics** is a GPU-accelerated, event-driven worker belonging to the **[NovaNature](https://www.google.com/search?q=https://nature.novaeco.tech)** sector.

To protect biodiversity, we must measure it. Traditional surveys (sending humans to count birds) are expensive and sporadic. This worker enables **Passive Acoustic Monitoring (PAM)** at scale. It ingests audio from field microphones, filters out noise (wind/rain), and uses AI to identify specific species—converting soundwaves into a "Biodiversity Index."

-----

## 🎯 Value Proposition

Biodiversity credits and reforestation projects require proof of life, not just proof of trees. This worker provides that proof:

1.  **Species Auditing:** Automatically logs the presence of indicator species (e.g., "Endangered Frog detected in Zone 4").
2.  **Threat Detection:** Identifies anthropogenic noise (chainsaws, gunshots, heavy machinery) to alert rangers of illegal logging or poaching.
3.  **Ecosystem Health:** Calculates "Soundscape Indices" (ACI, NDSI) to measure the overall complexity and health of a habitat over time.

-----

## 🏗️ Architecture (The Listening Pipeline)

This worker consumes the `queue.nature.audio-analysis` queue managed by NovaNature.

```mermaid
graph LR
    subgraph "The Forest (NovaInfra)"
        Sensor[Solar Microphone] -->|Upload .WAV| S3[(Object Storage)]
        Sensor -->|Event: New File| MQ[(RabbitMQ)]
    end

    subgraph "The Worker (NovaNature)"
        MQ -->|1. Consume Event| Worker[Bioacoustic Worker]
        Worker -->|2. Fetch Audio| S3
        Worker -->|3. Pre-process| DSP[DSP Engine (Spectrogram)]
        Worker -->|4. Inference| Mind[NovaMind (BirdNET/Rainforest Models)]
    end

    subgraph "Outputs"
        Worker -->|5. Species Log| Nature[NovaNature DB]
        Worker -->|6. Poaching Alert| Policy[NovaPolicy Alert]
    end
```

### The Analysis Lifecycle

1.  **Ingest:** Receives a job referencing an audio file URL (e.g., `s3://raw-audio/amazon-01/2026-11-01.wav`).
2.  **Pre-processing:** Uses Digital Signal Processing (DSP) to normalize volume, remove static, and slice the file into 3-second "windows."
3.  **Inference:** Sends the audio chunks (or generated spectrograms) to **NovaMind**.
      * *Query:* "Identify dominant species in this clip."
4.  **Classification:** Returns a list of taxa with confidence scores (e.g., `{"species": "Turdus migratorius", "confidence": 0.94}`).
5.  **Indexing:** Writes the observation to `NovaNature` geospatial database.

-----

## ✨ Key Features

### 1\. Multi-Taxa Identification

It doesn't just hear birds. It routes audio to specific models based on the sensor's metadata:

  * **Ultrasonic Sensors:** Routes to Bat identification models.
  * **Hydrophones:** Routes to Marine Mammal (whale/dolphin) models.
  * **Standard Mics:** Routes to Bird and Insect models.

### 2\. The "Chainsaw" Guard

A parallel classification process runs to detect "Threat Sounds."

  * If a chainsaw frequency pattern is detected in a protected area, it triggers a P1 Alert to **NovaPolicy** (Governance) and local rangers via **NovaInfra**.

### 3\. Soundscape Indices

Even if we can't identify the *exact* species, we can measure the *richness*.

  * **Acoustic Complexity Index (ACI):** Measures the variability of bird songs (high = healthy).
  * **Bio-phony vs. Anthro-phony:** Calculates the ratio of nature sounds vs. human machine noise.

-----

## 🚀 Getting Started

### Prerequisites

  * Docker Desktop (GPU support recommended for local inference testing)
  * Python 3.11+
  * **System Deps:** `ffmpeg`, `libsndfile1`

### Installation

1.  **Clone the repo:**
    ```bash
    git clone https://github.com/novaeco-tech/novanature-worker-bioacoustics.git
    cd novanature-worker-bioacoustics
    ```
2.  **Start the Dev Environment:**
    ```bash
    make dev
    ```
      * Starts the worker and a MinIO (S3 mock) container.
      * **Health Check:** `http://localhost:8080/health`

### Configuration (`.env`)

```ini
# Queue
RABBITMQ_URI=amqp://guest:guest@rabbitmq:5672/
QUEUE_NAME=queue.nature.audio-analysis

# AI Service
NOVAMIND_GRPC_URL=novamind-api:50051
MODEL_VERSION=birdnet-v2.4

# Storage
S3_ENDPOINT=http://minio:9000
S3_BUCKET=nature-audio
```

-----

## 📂 Repository Structure

```text
novanature-worker-bioacoustics/
├── src/
│   ├── main.py             # Consumer Loop
│   ├── audio/              # Librosa/FFmpeg wrappers for DSP
│   ├── indices/            # Math logic for ACI/NDSI calculations
│   └── clients/            # gRPC client for NovaMind
├── samples/                # Reference .wav files (Birdsong, Chainsaw)
├── tests/                  # Pytest suite
└── Dockerfile              # Python runtime with audio libraries
```

-----

## 🧪 Testing

We use **Fixture-Based Testing** with reference audio files.

  * **Spectrogram Test:** `make test-dsp`
      * Ensures that a `.wav` file is correctly converted to a Mel-spectrogram array.
  * **Integration Test:** `make test-pipeline`
      * Uploads `samples/robin.wav` to the mock bucket, triggers the worker, and asserts that the "American Robin" tag is returned.

-----

## 🤝 Contributing

We need contributors with backgrounds in **Signal Processing**, **Ecology**, and **Machine Learning**.
See [CONTRIBUTING.md](https://www.google.com/search?q=../.github/CONTRIBUTING.md) for details.

**Maintainers:** `@novaeco-tech/maintainers-sector-novanature`