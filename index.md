---
layout: default
---

This project demonstrates **how to detect and investigate brute force and malware attacks** using a cloud-based environment. The project was hosted on `Vultr`, where I set up an `ELK stack (Elasticsearch, Logstash, Kibana)`, performed _attacks_, and configured automated _alerts_ and _dashboards_ to monitor activity.

### Overview

*   **Platform: Vultr (Cloud)**
*   **Tools Used: ELK (Elasticsearch, Logstash, Kibana), Mythic C2, Sysmon, Ticketing System**
*   **Key Skills: SIEM setup, log ingestion, threat detection, incident response, dashboard creation, alert configuration**

# [1.ELK Setup and Sysmon Log Ingestion](#https://jeremiaheze.github.io/ELK.github.io/)

## ELK Stack Installation Confirmation
To set up the ELK stack, we installed Elasticsearch, Logstash, and Kibana on a Vultr server. These tools allow us to centralize, analyze, and visualize logs from multiple endpoints. Sysmon logs were collected to detect security incidents.

**Commands for ELK Installation:**

```bash
# Install Elasticsearch
sudo apt-get update
sudo apt-get install elasticsearch

# Install Logstash
sudo apt-get install logstash

# Install Kibana
sudo apt-get install kibana

# Start services
sudo systemctl start elasticsearch
sudo systemctl start logstash
sudo systemctl start kibana
```

## Sysmon Logs in Kibana

Sysmon (System Monitor) was deployed on endpoints to collect detailed system logs, which were then sent to Logstash for processing and forwarded to Elasticsearch for storage. Logs were visualized in Kibana dashboards.

**Sysmon Installation:**
```powershell
# Install Sysmon
Sysmon -accepteula -i sysmonconfig.xml

# Confirm Sysmon is running
Get-Service -Name Sysmon
```

**Logstash Configuration for Sysmon Logs:**
```bash
# /etc/logstash/conf.d/sysmon.conf
input {
  beats {
    port => 5044
  }
}
filter {
  xml {
    source => "message"
    target => "parsed"
  }
}
output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "sysmon-%{+YYYY.MM.dd}"
  }
}
```
## Logical Diagram
A logical diagram was created to showcase the data flow from endpoints (Sysmon) to the ELK stack. Data is collected, processed, stored, and visualized in Kibana.

# [2.Brute Force Attack Detection](https://jeremiaheze.github.io/BruteAttackDetect.github.io/)

## Kibana Alerts for Brute Force Attacks

To detect brute force attacks, we set up alerts in Kibana to monitor failed login attempts over a certain threshold. The detection was based on log patterns from SSH or RDP brute force attempts.

**Kibana Alert for SSH Brute Force:**
```json
{
  "trigger": {
    "source": "elasticsearch",
    "query": {
      "bool": {
        "must": [
          { "match": { "event.category": "authentication" }},
          { "range": { "event.outcome": { "gte": 10 }}}
        ]
      }
    }
  },
  "actions": [
    {
      "email": {
        "to": "admin@example.com",
        "subject": "Brute Force Attack Detected",
        "body": "Multiple failed SSH login attempts detected on {{trigger.time}}"
      }
    }
  ]
}
```
## Brute Force Activity Dashboard in Kibana
A custom dashboard was created in Kibana to visualize failed login attempts. This dashboard showed attack frequency, IP addresses involved, and success/failure rates.

# [3.Command and Control (C2) Attack Detection](https://jeremiaheze.github.io/C2Detect.github.io/)

## Mythic C2 Server Setup
For the Command and Control (C2) simulation, a Mythic C2 server was deployed to simulate communication between the attacker’s machine and compromised endpoints. This allowed us to generate malicious traffic for detection.

**Mythic C2 Installation:**
```bash
git clone https://github.com/its-a-feature/Mythic.git
cd Mythic
sudo ./start_mythic.sh
```
## C2 Attack Logs in Kibana
The ELK stack was configured to collect and monitor logs related to C2 communications. The data was processed and indexed into Elasticsearch, and alerts were set up to monitor suspicious outbound traffic patterns.

## Kibana Dashboard Visualizing C2 Activity
A dashboard was set up to visualize C2 activity by tracking unusual outbound connections, DNS requests, and data exfiltration attempts. This dashboard provided a detailed view of all suspicious C2-related activities.

# [4.Ticketing System Integration](https://jeremiaheze.github.io/TicketSystem.github.io/)

## Ticket Creation from Kibana Alerts
When a security alert was triggered (e.g., brute force or C2 activity), the system automatically created a ticket in the ticketing system to ensure the incident was tracked and handled by the security team.

**Sample Kibana Alert Configuration (automating ticket creation):**
```json
{
  "trigger": {
    "source": "elasticsearch",
    "query": {
      "match": { "event.category": "network" }
    }
  },
  "actions": [
    {
      "webhook": {
        "method": "POST",
        "url": "http://ticketingsystem.example.com/api/tickets",
        "body": {
          "ticket_type": "security_alert",
          "message": "C2 activity detected on {{trigger.time}}"
        }
      }
    }
  ]
}
```
## Ticketing System Interface with Active Incident Tickets
The ticketing system displayed a list of active tickets, categorized by priority, type, and status. Each ticket contained detailed information about the alert, along with suggested investigation steps.




Text can be **bold**, _italic_, ~~strikethrough~~ or `keyword`.

[Link to another page](./another-page.html).

There should be whitespace between paragraphs.

There should be whitespace between paragraphs. We recommend including a README, or a file with information about your project.

# Header 1

This is a normal paragraph following a header. GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere.

## Header 2

> This is a blockquote following a header.
>
> When something is important enough, you do it even if the odds are not in your favor.

### Header 3

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

### Large image

![Branching](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```
