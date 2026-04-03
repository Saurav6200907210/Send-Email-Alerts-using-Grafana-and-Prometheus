# 🚀 Monitor Express.js App with Prometheus + Grafana & Email Alerts

A simple and beginner-friendly guide to monitor your **Express.js application** using **Prometheus + Grafana**, and receive **real-time email alerts** when requests come in 🚨📧

---

## 📌 Overview

This setup allows you to:

* 📊 Collect metrics from Express.js
* ⚡ Scrape metrics using Prometheus
* 📈 Visualize data in Grafana
* 📧 Receive email alerts on activity

---

## ⚡ Step 1: Initialize Node.js Project

```bash id="init123"
mkdir express-prometheus-grafana
cd express-prometheus-grafana
npm init -y
npm install express prom-client
```

---

## ⚡ Step 2: Create Express Server with Metrics

Create a file `server.js`:

```javascript id="server321"
const express = require("express");
const client = require("prom-client");

const app = express();
const PORT = 3001;

// Registry
const register = new client.Registry();
client.collectDefaultMetrics({ register });

// Custom metric: request counter
const httpRequestCount = new client.Counter({
  name: "http_request_count",
  help: "Total number of HTTP requests",
});
register.registerMetric(httpRequestCount);

// Middleware
app.use((req, res, next) => {
  httpRequestCount.inc();
  next();
});

// Route
app.get("/", (req, res) => {
  res.send("Hello Prometheus + Grafana!");
});

// Metrics endpoint
app.get("/metrics", async (req, res) => {
  res.set("Content-Type", register.contentType);
  res.end(await register.metrics());
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

---

### ▶️ Run the Server

```bash id="run321"
node server.js
```

👉 Open: `http://localhost:3001/metrics`
You should see Prometheus metrics ✅

---

## ⚡ Step 3: Configure Prometheus

```bash id="promcfg"
sudo nano /etc/prometheus/prometheus.yml
```

Add:

```yaml id="promyaml"
scrape_configs:
  - job_name: "express-app"
    static_configs:
      - targets: ["<your-system-ip>:3001"]
```

👉 Replace `<your-system-ip>` with your IP
(e.g., `192.168.1.100:3001`)

---

### 🔄 Restart Prometheus

```bash id="restartprom"
sudo systemctl restart prometheus
```

👉 Verify: `http://localhost:9090/classic/targets`
Status should be **UP ✅**

---

## ⚡ Step 4: Configure SMTP in Grafana

```bash id="grafanaedit"
sudo nano /etc/grafana/grafana.ini
```

Update:

```ini id="smtpblock"
[smtp]
enabled = true
host = smtp.gmail.com:587
user = your-email@gmail.com
password = "your-app-password"
from_address = your-email@gmail.com
from_name = Grafana
startTLS_policy = OpportunisticStartTLS
```

---

### 🔄 Restart Grafana

```bash id="restartgrafana"
sudo systemctl restart grafana-server
sudo systemctl status grafana-server
```

---

## ⚡ Step 5: Create Contact Point

* Open Grafana UI → **Alerting → Contact Points**
* Click **New Contact Point**
* Type: Email
* Name: `My Email`
* Recipients: `your@domain.com`

✅ (Optional: Send test email)

---

## ⚡ Step 6: Create Dashboard & Alert

* Go to **Dashboards → New Dashboard → Add Panel**

### 🔹 Query:

```bash id="query123"
rate(http_request_count[1m])
```

### 🔹 Alert Setup:

* Enable Alert
* Condition: `A > 0` (means request detected)
* Notification: Select **My Email**

Click **Save Dashboard ✅**

---

## ✅ Final Result

* Express app exposes metrics
* Prometheus scrapes metrics
* Grafana visualizes request rate
* Email alert triggers on request

---

## 📊 Example Flow

* Open: `http://localhost:3001/`
* Request count increases 📈
* Grafana dashboard shows spike
* Email alert received 🚨

**Example Alert:**
`Alert firing: Express App Request Alert – at least 1 request detected!`

---

## 🎉 Conclusion

You have successfully built a **full monitoring system**:

* Express.js + Metrics
* Prometheus + Scraping
* Grafana + Visualization
* Email Alerts 🚀

---

## 🌟 Author

Made with Saurav❤️

---

⭐ If this helped you, don’t forget to **star the repo**!

**Happy Monitoring 🚀**
