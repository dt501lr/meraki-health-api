
<img src="img/meraki_health_check.png">

[![published](https://static.production.devnetcloud.com/codeexchange/assets/images/devnet-published.svg)](https://developer.cisco.com/codeexchange/github/repo/obrigg/Vanilla-ISE)

### The Challenge

Cisco Meraki is an amazing cloud-managed IT solution, simplying network, security, security cameras and IoT infrastructure.
However, even the most intelligent AI/ML driven solution is still volunerable to users misconfiguring various options (usually without reading the documentation). Misconfiguration can result in an outage, or poor user experience (if you will limit user's traffic to 1Mbps - things will work slowly.. AI won't help there as it's the admin's "intent").

### The Solution

This script will leverage the Meraki API to compare an organization's settings and status against a set of best practices and thresholds - uncovering configurations that should be changed.

#### Example output
Summary tab:
<p align="center"><img src="img/summary.png"></p>
Channel utilization tab:
<p align="center"><img src="img/rf_utilization.png"></p>
RF Profile tab:
<p align="center"><img src="img/rf_profile.png"></p>
Switchport counters tab:
<p align="center"><img src="img/switch_port_counters.png"></p>

---
### List of checks
#### General checks
1. Network heath alerts: gathering all health alerts from all of the organization's networks.
2. Multiple adminstrator users: verifying the organization has more than one admin with full control (per best practices).
3. Admin 2FA: checks which admin users have 2FA enabled, and which admins do not (per best practices).
4. API calls: present whether Dashboard API is being used, and by which admin usesr.
5. API v0 usage: The Dashboard API v0 is being deprecated, and integrations should be updated to API v1.
6. Firmware checks: compares the firmware versions of each network and device types to the latest stable release.
#### Wireless checks
1. Channel utilization (for 5GHz only, 2.4GHz is beyond saving...)
2. RF Profile check:
    * Configured Minimum Tx power (usually mistaken with EIRP, resulting to too high Tx power).
    * Configured minimum Bitrate (see [best practices](https://documentation.meraki.com/MR/WiFi_Basics_and_Best_Practices/Multi-SSID_Deployment_Considerations)).
    * Configured channel Width.
    * Manually configured RX-SOP (most won't configure it right, and it's better left at "auto").
    * Number of enabled SSIDs (see [best practices](https://documentation.meraki.com/MR/WiFi_Basics_and_Best_Practices/Multi-SSID_Deployment_Considerations)).

#### Switching checks
1. Are jumbo-frames enabled, by checking the MTU (see [best practices](https://documentation.meraki.com/Architectures_and_Best_Practices/Cisco_Meraki_Best_Practice_Design/Best_Practice_Design_-_MS_Switching/General_MS_Best_Practices)).
2. Is RSTP enabled? (best of luck handling loops without it.. see [best practices](https://documentation.meraki.com/Architectures_and_Best_Practices/Cisco_Meraki_Best_Practice_Design/Best_Practice_Design_-_MS_Switching/General_MS_Best_Practices))
3. Port counters:
    * CRC errors.
    * Collisions.
    * Broadcasts exceeding threshold.
    * Multicasts exceeding threshold.
    * Topology changes (TCNs) exceeding threshold.
---
Convinced the health-check is worth 5 minutes of your time? let's do this!
### How to run the script:

#### Generate your Meraki API Key

1. Access the [Meraki dashboard](dashboard.meraki.com).
2. For access to the API, first enable the API for your organization under Organization > Settings > Dashboard API access.
<p align="center"><img src="img/org_settings.png"></p>
3. After enabling the API, go to "my profile" on the upper right side of the dashboard to generate an API key. This API key will be associated with the Dashboard Administrator account which generates it, and will inherit the same permissions as that account.  You can generate, revoke, and regenerate your API key on your profile.
<p align="center"><img src="img/my_profile.png"></p>
<p align="center"><img src="img/api_access.png"></p>
**Always keep your API key safe as it provides authentication to all of your organizations with the API enabled. If your API key is shared, you can regenerate your API key at any time. This will revoke the existing API key.**

#### Storing the Meraki API Key as an environment variable
Once the API key is obtained, you'll need to store the Meraki dashboard API key as an environment variable:
`export MERAKI_DASHBOARD_API_KEY = <YOUR MERAKI API KEY>`
and install the Meraki SDK via `pip install -r requirements.txt`

Now you're ready. Good luck!

`python async_run.py`

### Known limitations / caveats
1. The script intentionally ignores the 2.4GHz spectrum, as it is beyond salavion. It can be altered, if needed, in the `check_wifi_channel_utilization` function.
2. The Meraki API **does not retrieve the default RF policies**. A network using a default RF policy with altered values will not show up in the report.
3. The SSID amount check counts every **enabled** SSIDs, even if the SSID is limited to certain APs or to a certain band. You may have three ssids on 2.4GHz and three different SSIDs on 5GHz, but the check will fail as it counts six SSIDs.
4. The API usage is checking the last 5,000 API calls. It can be changed in the code, more API calls being examines = longer run time for the script (The async version checks up to 10,000 API calls per admin user).
----
### Licensing info
Copyright (c) 2022 Cisco and/or its affiliates.

This software is licensed to you under the terms of the Cisco Sample
Code License, Version 1.1 (the "License"). You may obtain a copy of the
License at

               https://developer.cisco.com/docs/licenses

All use of the material herein must be in accordance with the terms of
the License. All rights not expressly granted by the License are
reserved. Unless required by applicable law or agreed to separately in
writing, software distributed under the License is distributed on an "AS
IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
or implied.
