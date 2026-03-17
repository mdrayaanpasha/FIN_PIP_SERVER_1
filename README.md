## OHLCV Data Producer (`s_1`)

This service is a **Scheduled Event Producer**. It acts as a bridge that pulls financial market data, saves it to a fast cache (Redis), and then tells other services that new data is ready via Kafka.

---

### 📥 Data Flow
The service follows a simple **Pull → Cache → Notify** pattern every 60 minutes:



1.  **Pull:** It fetches the latest OHLCV (Market) data from an external source.
2.  **Cache:** It saves that data into **Redis** for 10 minutes so other services can read it instantly without re-fetching.
3.  **Notify:** It sends a message to three **Kafka Topics** (`anomaly`, `indicator`, `persistence`) to alert downstream services.

---

### 🛠️ What it Needs (Requirements)
To keep the system secure and fast, you need:

* **Node.js 20+**: The engine running the code.
* **Redis**: For high-speed temporary storage.
* **Kafka Cluster**: To handle the messaging between services.
* **SSL Certificates**: To keep the connection to Kafka encrypted.

---

### 🔑 Setup (Environment Variables)
Create a `.env` file with these keys. The service uses **Base64** strings for the certificates to make them easier to pass through cloud environments.

| Variable | What it is |
| :--- | :--- |
| `KAFKA_URL` | The "address" of your Kafka broker. |
| `REDIS_URL` | The "address" of your Redis database. |
| `SERVICE_CERT` | Your SSL Certificate (Base64 string). |
| `SERVICE_KEY` | Your Private Key (Base64 string). |

---

### 🐳 Run with Docker
The project includes a **Dockerfile** for easy deployment. It uses a lightweight "Alpine" version of Node.js to keep the file size small.

**1. Build the image:**
```bash
docker build -t ohlcv-producer .
```

**2. Start the service:**
```bash
docker run --env-file .env ohlcv-producer
```

---

### ⏱️ The Automated Logic
You don't need to trigger this service manually. It uses a **CRON job** that runs automatically every **1 minute**. 

* **Topic Broadcast:** It sends a small message (the string `"redis"`) to the topics. This acts as a "pointer"—it tells other services, *"Hey, the data is updated in Redis, go look there!"*
