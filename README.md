# NewRelic
New Relic is a cloud-based observability platform that helps developers and IT operations teams monitor, debug, and improve the performance of their applications and infrastructure. It provides real-time insights through features like application performance monitoring (APM), infrastructure monitoring, log management, and distributed tracing. 

Here’s a polished `README.md` template for installing New Relic on Ubuntu 24.04, setting up APM monitoring, creating alerts, and writing NRQL queries. Feel free to adapt it to your project.

---

# 🚀 New Relic Setup & Monitoring Guide  
A step-by-step walkthrough for installing New Relic on **Ubuntu 24.04**, configuring **APM**, creating **alerts**, and using **NRQL** for custom queries.

---

## 🎯 Prerequisites
- Ubuntu 24.04 server instance with `sudo` access  
- An active New Relic account + **License Key**  
- Running application (e.g., Node.js, Java, Python, etc.) on the server

---

## 1. Install the New Relic Infrastructure Agent
The agent collects host-level metrics and system data.

```bash
# Update OS
sudo apt update && sudo apt upgrade -y

# Add New Relic's repository and install
echo "deb [arch=amd64] https://download.newrelic.com/infrastructure_agent/linux/apt jammy main" | sudo tee /etc/apt/sources.list.d/newrelic-infra.list
curl -s https://download.newrelic.com/infrastructure_agent/gpg/newrelic-infra.gpg | sudo apt-key add -
sudo apt update
sudo apt install newrelic-infra -y

# Configure with your license key
sudo bash -c 'echo "license_key: YOUR_LICENSE_KEY" >> /etc/newrelic-infra.yml'

# Start & enable service
sudo systemctl enable newrelic-infra
sudo systemctl start newrelic-infra
````

✅ The agent should now report host metrics to New Relic.

---

## 2. Install APM for Your Application

Depending on your application stack:

### Node.js

```bash
npm install newrelic --save
# Add at top of app’s main file (e.g., index.js):
require('newrelic');
```

### Python (Gunicorn/Flask/Django)

```bash
pip install newrelic
newrelic-admin generate-config YOUR_LICENSE_KEY newrelic.ini
# Start your app with:
NEW_RELIC_CONFIG_FILE=newrelic.ini newrelic-admin run-program gunicorn myapp:app
```

### Java (Spring, etc.)

* Download the Java agent (`newrelic-java.zip`) from your New Relic dashboard.
* Unzip in your application directory.
* Add to JVM launch:

  ```bash
  -javaagent:/path/to/newrelic/newrelic.jar
  ```
* The default `newrelic.yml` is auto-configured with your license key.

---

## 3. Verify APM Data in Dashboard

* Log in to New Relic UI → **APM**
* Verify your app appears and it’s reporting metrics: response times, throughput, error rates.

---

## 4. Create an Alert Policy + NRQL-Based Alert

Using a custom NRQL query:

### A. Define the NRQL query

Example: high error rate over 5 minutes

```sql
SELECT count(*) FROM TransactionError WHERE appName = 'MyApp' AND errorType IS NOT NULL SINCE 5 minutes AGO
```

### B. Create Alert Policy

1. In New Relic, go to **Alerts & AI → Policies**
2. Click **Create a policy** → name it (e.g., "MyApp Production Alerts")
3. **Add condition** → choose **NRQL**
4. Inject the query
5. Set **Threshold**: e.g., `count > 5` for at least 5 minutes
6. Assign **Notification channels** (email, Slack, PagerDuty, etc.)

---

## 5. Example NRQL Queries

* **Slow transactions**:

  ```sql
  SELECT percentile(duration, 95) FROM Transaction WHERE appName='MyApp' SINCE 1 hour AGO
  ```
* **High CPU usage**:

  ```sql
  SELECT average(cpuPercent) FROM SystemSample WHERE hostname = 'my-ubuntu-server' SINCE 5 minutes AGO
  ```
* **Database error spikes**:

  ```sql
  SELECT count(*) FROM DatastoreError WHERE appName = 'MyApp' FACET databaseCall
  ```

---

## 6. Dashboarding & Visualization

* Navigate to **Dashboards** → **Create**
* Add charts using NRQL
* Example widget:

  ```sql
select average(duration) FROM Transaction WHERE appName='MyApp'
  
select average(newrelic.logs.batchIndex) FROM Log

select min(newrelic.logs.batchIndex) from Log 

select max(newrelic.logs.batchIndex) from Log

select trace.id from Transaction where timestamp >= 10

select * from MysqlSample where apmApplicationNames = '|app1|' and timestamp = 1751554763000

select * from Transaction limit 10

select * from Transaction since 1 day ago LIMIT 10

select * from Transaction since 1 day ago TIMESERIES 

select * from Transaction where transactionSubType='Filter' SINCE 5 days ago TIMESERIES 1 hour 

SELECT count(*) from Transaction FACET transactionSubType SINCE 7 days ago limit 3

select count(entity.guid) from TransactionError

select uniqueCount(entity.guid) from TransactionError

select latest(entity.guid) from TransactionError

---

## ✅ Summary Checklist

| Step | Task                                 |
| ---- | ------------------------------------ |
| ✅    | Install Infrastructure Agent         |
| ✅    | Install APM agent for your stack     |
| ✅    | Confirm APM data in UI               |
| ✅    | Create Alert Policy & NRQL condition |
| ✅    | Add Notification Channel             |
| ✅    | Create NRQL-powered dashboards       |

---

## 🔗 Helpful Resources

* [New Relic Agents Documentation](https://docs.newrelic.com/docs/agents)
* [NRQL Query Syntax Guide](https://docs.newrelic.com/docs/query-your-data/nrql-new-relic-query-language/get-started/nrql-syntax-clauses/)
* [Alerting Policies & Conditions](https://docs.newrelic.com/docs/alerts-applied-intelligence)

---

