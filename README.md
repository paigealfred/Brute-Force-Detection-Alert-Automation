# SSH Brute Force Detection & Alert Automation

## 🎯 Project Overview

This Python-based security automation tool monitors authentication logs for SSH brute force attacks, automatically detects suspicious activity based on configurable thresholds, and generates severity-based alerts for SOC analysts. The tool processes authentication events, tracks failed login attempts per user/IP combination, and exports actionable alerts to CSV format for incident response workflows.


## 📸 Screenshots

### Authentication Logs Input
![Auth Logs](1_auth_logs_input.png)

### Console Detection Output
![Detection Results](3_detection_output.png)

### Detection Script
![Python Script](2_detection_script.png)

### Alert CSV Export
![CSV Output](4_alerts_csv.png)


## 🔍 Use Case

In production SOC environments, analysts face thousands of authentication events daily. Manual review is time-intensive and prone to human error. This automation:
- **Reduces alert fatigue** by filtering noise and surfacing only high-confidence threats
- **Accelerates incident response** through automatic severity classification
- **Enables proactive threat hunting** by tracking failed authentication patterns
- **Provides audit trails** with structured CSV output for compliance reporting

## 🛠️ Technical Stack

- **Language:** Python 3.x
- **Libraries:** 
  - `csv` - Authentication log parsing and alert export
  - `collections.defaultdict` - Efficient failed attempt tracking per user/IP pair

## 📋 Features

✅ **Automated Log Parsing** - Ingests CSV-formatted authentication logs  
✅ **Brute Force Detection** - Identifies repeated failed login attempts from unique IP addresses  
✅ **Configurable Threshold** - Customizable alert trigger (default: 5 failed attempts)  
✅ **Severity Classification** - Assigns "high" (10+ failures) or "medium" (5-9 failures) risk levels  
✅ **CSV Alert Export** - Generates actionable alerts with username, source IP, failure count, and severity  
✅ **Scalable Design** - Handles large log volumes with dictionary-based tracking  

## 🚀 How It Works

### Detection Logic

1. **Log Ingestion**: Reads `auth_logs.csv` containing authentication events
2. **Failed Login Tracking**: Monitors only `FAILED` status events, ignoring successful authentications
3. **Aggregation**: Groups failed attempts by `(username, source_IP)` pairs using `defaultdict`
4. **Threshold Comparison**: Flags users/IPs exceeding 5 failed attempts (configurable)
5. **Severity Assignment**: 
   - **High**: ≥10 failed attempts (indicates aggressive brute force)
   - **Medium**: 5-9 failed attempts (potential reconnaissance or weak attack)
6. **Alert Generation**: Exports findings to `bruteforce_alerts.csv` for analyst review

### Sample Detection

**Input Log (`auth_logs.csv`):**
```
timestamp,username,src_ip,status
2025-11-15T10:08:01Z,alice,10.0.0.5,FAILED
2025-11-15T10:08:05Z,alice,10.0.0.5,SUCCESS
2025-11-15T11:05:01Z,carol,172.16.0.3,FAILED
2025-11-15T11:05:05Z,carol,172.16.0.3,FAILED
2025-11-15T11:05:10Z,carol,172.16.0.3,FAILED
2025-11-15T11:05:15Z,carol,172.16.0.3,FAILED
2025-11-15T11:05:20Z,carol,172.16.0.3,FAILED
```

**Output Alert (`bruteforce_alerts.csv`):**
```
username,src_ip,fail_count,severity
carol,172.16.0.3,5,medium
```

## 📂 Project Structure

```
soc_automation/
│
├── auth_logs.csv              # Input: Authentication event logs
├── bruteforce_detector.py     # Main detection script
└── bruteforce_alerts.csv      # Output: Generated security alerts
```

## 🔧 Installation & Usage

### Prerequisites
```bash
Python 3.7 or higher
```

### Setup
```bash
# Clone the repository
git clone https://github.com/paigealfred/soc-bruteforce-automation.git
cd soc-bruteforce-automation

# Ensure auth_logs.csv is in the same directory
# Or modify LOG_FILE variable in bruteforce_detector.py
```

### Run Detection
```bash
python bruteforce_detector.py
```

### Expected Output
```
[ALERT] Possible brute force on user carol from 172.16.0.3 - 5 failed logins
Saved 1 alert(s) to bruteforce_alerts.csv
```

## 📊 Configuration

### Modify Detection Threshold

Edit `bruteforce_detector.py`:
```python
FAIL_THRESHOLD: int = 5  # Change to desired failed attempt count
```

### Adjust Severity Levels

Modify lines 30-31:
```python
'severity': 'high' if count >= (FAIL_THRESHOLD * 2) else 'medium'
# Example: Change multiplier for stricter/looser classification
```

### Custom Log File Path

Update line 4:
```python
LOG_FILE: str = "auth_logs.csv"  # Change to your log file location
```

## 🧠 Code Breakdown

### Line-by-Line Explanation

```python
import csv
from collections import defaultdict
```
**Purpose:** Import required libraries  
- `csv`: Provides CSV file reading/writing capabilities for structured log data
- `defaultdict`: Auto-initializes dictionary values to `int(0)`, eliminating KeyError exceptions when tracking failed attempts

---

```python
LOG_FILE: str = "auth_logs.csv"
```
**Purpose:** Define input log file path  
**Why Needed:** Centralizes file location for easy modification without changing core logic

---

```python
FAIL_THRESHOLD: int = 5
```
**Purpose:** Set brute force detection threshold  
**Why Needed:** Configurable trigger point balancing false positives (too low) vs. missed attacks (too high)

---

```python
failed_attempts = defaultdict(int)
```
**Purpose:** Initialize tracking dictionary for failed login counts  
**How It Works:** 
- Key: `(username, src_ip)` tuple
- Value: Integer count of failures
- `defaultdict(int)` returns `0` for new keys, avoiding `if key exists` checks

---

```python
with open(LOG_FILE, "r") as f:
    reader = csv.DictReader(f)
```
**Purpose:** Open log file in read mode and create CSV parser  
**Why `DictReader`:** Returns each row as a dictionary (`{'username': 'alice', ...}`), enabling intuitive column access vs. index-based parsing

---

```python
for row in reader:
```
**Purpose:** Iterate through each authentication event  
**Why Needed:** Process logs sequentially to aggregate failure counts

---

```python
status = row.get('status', '').upper()
username = row.get('username', '')
src_ip = row.get('src_ip', '')
```
**Purpose:** Extract and normalize event fields  
**Why `.get()` Method:** Safely handles missing columns, returning empty string instead of raising KeyError  
**Why `.upper()`:** Ensures case-insensitive status matching ('failed' vs. 'FAILED')

---

```python
if status == 'FAILED':
```
**Purpose:** Filter only failed authentication attempts  
**Why Needed:** Ignore successful logins to focus on brute force indicators

---

```python
key = (username, src_ip)
failed_attempts[key] += 1
```
**Purpose:** Increment failure count for this user/IP combination  
**Why Tuple Key:** Tracks per-source attacks on specific accounts (not just total failures per IP)  
**Example:** `('admin', '10.0.0.5')` counted separately from `('root', '10.0.0.5')`

---

```python
alerts = []
```
**Purpose:** Initialize list to store detected threats  
**Why List:** Allows appending multiple alerts for batch CSV export

---

```python
for (username, src_ip), count in failed_attempts.items():
```
**Purpose:** Iterate through aggregated failure counts  
**`.items()` Method:** Returns both key (`username, src_ip`) and value (`count`) for processing

---

```python
if count >= FAIL_THRESHOLD:
```
**Purpose:** Apply detection threshold  
**Why `>=`:** Inclusive comparison catches exactly 5 failures (not just 6+)

---

```python
print(f"[ALERT] Possible brute force on user {username} from {src_ip} - {count} failed logins")
```
**Purpose:** Real-time console notification for SOC analysts  
**Why f-string:** Clean, readable output with variable interpolation

---

```python
alerts.append({
    'username': username,
    'src_ip': src_ip,
    'fail_count': count,
    'severity': 'high' if count >= (FAIL_THRESHOLD * 2) else 'medium'
})
```
**Purpose:** Build alert object with structured data  
**Severity Logic:**
- **High:** ≥10 failures (2× threshold) indicates sustained attack
- **Medium:** 5-9 failures suggests reconnaissance or weak credentials
**Why Dictionary:** Matches CSV fieldnames for DictWriter export

---

```python
if alerts:
```
**Purpose:** Check if any threats were detected  
**Why Needed:** Prevents creating empty CSV files when no alerts triggered

---

```python
output_file = 'bruteforce_alerts.csv'
fieldnames = ['username', 'src_ip', 'fail_count', 'severity']
```
**Purpose:** Define export configuration  
**`fieldnames` List:** Specifies CSV column order and headers

---

```python
with open(output_file, 'w', newline='') as f:
```
**Purpose:** Open output file in write mode  
**`newline=''` Parameter:** Prevents extra blank rows in Windows environments (CSV module requirement)

---

```python
writer = csv.DictWriter(f, fieldnames=fieldnames)
```
**Purpose:** Create CSV writer with defined columns  
**`DictWriter`:** Maps dictionary keys to CSV columns automatically

---

```python
writer.writeheader()
```
**Purpose:** Write column names as first row  
**Example Output:** `username,src_ip,fail_count,severity`

---

```python
writer.writerows(alerts)
```
**Purpose:** Write all alert dictionaries to CSV  
**Batch Operation:** More efficient than writing one row at a time

---

```python
print(f"\nSaved {len(alerts)} alert(s) to {output_file}")
```
**Purpose:** Confirm successful export with alert count  
**`\n` Character:** Adds blank line for readability

---

```python
else:
    print('\nNo brute-force alerts found.')
```
**Purpose:** Notify analyst when no threats detected  
**Why Important:** Confirms script ran successfully even with clean logs

## 🎓 Learning Outcomes

This project demonstrates:
- **Log Analysis Automation** - Parsing structured security logs programmatically
- **Threat Detection Logic** - Implementing SIEM-like correlation rules in Python
- **Data Aggregation** - Using dictionaries for efficient event grouping
- **Incident Response Workflows** - Generating actionable alerts with severity classification
- **Production-Ready Code** - Error handling, configurability, and clear output formatting

## 🔐 Security Considerations

⚠️ **Threshold Tuning**: Default threshold (5 failures) may require adjustment based on:
- Environment size (higher thresholds for large user bases)
- Authentication mechanisms (MFA reduces false positives)
- Attack surface (internet-facing SSH requires stricter thresholds)

⚠️ **False Positives**: Legitimate scenarios triggering alerts:
- Users genuinely forgetting passwords
- Automated systems with misconfigured credentials
- Time zone differences causing repeated login attempts

⚠️ **Performance**: For logs exceeding 100K events, consider:
- Streaming log processing instead of loading entire file into memory
- Database storage for failed_attempts tracking
- Parallel processing for multi-threaded analysis

## 📈 Future Enhancements

- [ ] **Time-Based Analysis**: Detect concentrated attacks within specific time windows
- [ ] **IP Reputation Integration**: Cross-reference source IPs with threat intelligence feeds
- [ ] **Automated Blocking**: Generate firewall rules or SIEM actions for confirmed threats
- [ ] **Dashboard Integration**: Export alerts to Splunk/Elastic via API
- [ ] **Machine Learning**: Anomaly detection for sophisticated, low-and-slow attacks

## 📝 Author

