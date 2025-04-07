# ClouDNS DDNS Update Documentation

Below is a brief documentation that explains how to update your A-record using ClouDNS’s API. The documentation is structured into sections for finding the record ID and for updating the record with your current public IP address.


1. Find Your Record ID

To update a specific DNS record, you first need to retrieve its Record ID. Run the following command in your terminal:

```bash
curl -X GET "https://api.cloudns.net/dns/records.json?auth-id=35392&auth-password=Gacf3hkLaGp7mUT2gMEBEHzu&domain-name=domain.com&host=support&type=a&rows-per-page=10&page=1"
```

Expected Output

The API will return a JSON object containing the details of the DNS records for the specified domain and host. For example:

> {"592120637":{"id":"592120637","type":"A","host":"support","record":"4.3.2.4","dynamicurl_status":0,"failover":"0","ttl":"3600","status":1}}

In this example, the Record ID is 592120637. This is the unique identifier for the A-record corresponding to support.domain.com.

⸻

1. Update the A-Record

Once you have the Record ID, you can update the record with your current public IP. Use the following URL structure to update the record via ClouDNS API:

```bash
#!/bin/bash
# Aktuelle öffentliche IPv4-Adresse abrufen
ip=$(curl -s https://api.ipify.org)

# ClouDNS A-Record aktualisieren (support.domainc.com -> $ip)
curl -s "https://api.cloudns.net/dns/mod-record.json?auth-id=35392&auth-password=Gacf3hkLaGp7mUT2gMEBEHzu&domain-name=domain.com&record-id=592120637&host=support&record=$ip&ttl=3600"
```

⸻

3. Complete Update Script

Below is the complete Bash script that automatically retrieves your current public IP address and updates the A-record for support.domain.com.

#!/bin/bash
# This script updates the A-record for support.domain.com with your current public IP.

# Retrieve current public IP using ipify

ip=$(curl -s https://api.ipify.org)

# Update the A-record via ClouDNS API
curl -s "https://api.cloudns.net/dns/mod-record.json?auth-id=35392&auth-password=Gacf3hkLaGp7mUT2gMEBEHzu&domain-name=domain.com&record-id=592098172&host=support&record=$ip&ttl=3600"

How It Works
	1.	Retrieve Public IP:
The script uses curl to request your public IP from ipify.
	2.	Update the DNS Record:
It then calls the ClouDNS API endpoint mod-record.json with all the necessary parameters:
- auth-id and auth-password: Your ClouDNS API credentials.
- domain-name: The domain of the record (domain.com).
- record-id: The unique identifier of the record to update (592098172).
- host: The subdomain part of the record (support).
- record: The new IP address retrieved from ipify.
- ttl: Time to Live for the DNS record (set to 3600 seconds).

⸻

Usage
1.	Save the Script:
	Save the complete update script as update_dns.sh.
2.	Make It Executable:
	Run the command:

```bash
chmod +x update_dns.sh
```

3.	Run the Script:
	Execute the script by running:

```bash
./update_dns.sh
```

This script will update your A-record with the current public IP automatically. You can schedule it with cron for regular updates if needed.
