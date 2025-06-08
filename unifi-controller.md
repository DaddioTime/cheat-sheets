# UniFi - Controller installation

## ufw (Firewall)
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

sudo ufw allow 8080/tcp
sudo ufw allow 8443/tcp
sudo ufw allow 8843/tcp
sudo ufw allow 3478/udp
```

## Configure Time Zone

Set up your Time Zone
```bash
sudo dpkg-reconfigure tzdata
```

## Configure NTP
Set up NTP (Network Time Protocol)

```bash
sudo apt-get update
sudo apt-get install ntp
```

This sets up your time server and syncs the clock.  You can check NTP status by running:

```bash
service ntp status
```
Your local time should be set correctly (in 24-hour format).


## Set Up a Swap File 

Run the following commands to create and use a swap file.

```bash
sudo fallocate -l 4G /swapfile
```

This allocates 4GB as a swapfile in the /swapfile directory.

```bash
sudo chmod 600 /swapfile
```

Sets permissions on the swap file directory.

```bash
sudo mkswap /swapfile
```

Formats the swap file directory for use as a swap file.

```bash
sudo swapon /swapfile
```

Tells the server to use that directory as a swap file.
Finally, we need to make sure that the swap file turns on every time we reboot the server.  Run the following command to add the swap file information to the /etc/fstab file:

```bash
sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'
```

To verify that the command worked, type:

```bash
cat /etc/fstab
```

and look for a line that says:

```bash
/swapfile none swap sw 0 0
```

If that line exists in the /etc/fstab file, you’re all good.


## UniFi install

```bash
sudo apt-get update; apt-get install ca-certificates curl -y
sudo curl -sO https://get.glennr.nl/unifi/install/unifi-9.0.108.sh
sudo bash unifi-8.5.6.sh
```

> Source Script: https://glennr.nl/s/unifi-network-controller


```
https://<ip-address>:8443
```

## Let’s Encrypt

### Installation
```bash
#!/bin/bash

# Variablen
DOMAIN="unifi.domain.com"
CERT_DIR="/etc/letsencrypt/live/$DOMAIN"
UNIFI_DIR="/var/lib/unifi"
UNIFI_KEYSTORE="$UNIFI_DIR/keystore"
UNIFI_SERVICE="unifi"

# Certbot verwenden, um das Zertifikat zu erneuern (falls erforderlich)
echo "Erneuern des Let's Encrypt-Zertifikats..."
certbot certonly --standalone --preferred-challenges http -d $DOMAIN --renew-by-default

# Sicherstellen, dass die Zertifikate existieren
if [ ! -f "$CERT_DIR/fullchain.pem" ] || [ ! -f "$CERT_DIR/privkey.pem" ]; then
    echo "Fehler: Zertifikate wurden nicht gefunden."
    exit 1
fi

# Key und Zertifikat zusammenführen
echo "Zusammenführen von Key und Zertifikat..."
cat "$CERT_DIR/privkey.pem" "$CERT_DIR/fullchain.pem" > "$CERT_DIR/combined.pem"

# UniFi keystore Backup erstellen
echo "Erstellen eines Backups der UniFi Keystore..."
cp "$UNIFI_KEYSTORE" "$UNIFI_KEYSTORE.bak"

# PKCS12 Datei aus PEM Datei erstellen
echo "Erstellen einer PKCS12-Datei aus PEM-Datei..."
openssl pkcs12 -export -in "$CERT_DIR/combined.pem" -out "$CERT_DIR/unifi.p12" -name unifi -password pass:aircontrolenterprise

# Keystore aktualisieren
echo "Aktualisieren des UniFi Keystore..."
keytool -importkeystore -deststorepass aircontrolenterprise -destkeypass aircontrolenterprise -destkeystore "$UNIFI_KEYSTORE" -srckeystore "$CERT_DIR/unifi.p12" -srcstoretype PKCS12 -srcstorepass aircontrolenterprise -alias unifi -noprompt

# Berechtigungen setzen
echo "Setzen der Berechtigungen..."
chown unifi:unifi "$UNIFI_KEYSTORE"
chmod 640 "$UNIFI_KEYSTORE"

# UniFi Controller neu starten
echo "Neustarten des UniFi Controllers..."
systemctl restart $UNIFI_SERVICE

echo "Der Prozess ist abgeschlossen."
```

### Run Script

```bash
sudo apt-get install certbot
sudo certbot certonly --standalone --preferred-challenges http -d unifi.domain.com

chmod +x update_unifi_cert.sh
sudo ./update_unifi_cert.sh
```

### Cronjob for Let’s Encrypt

```bash
sudo crontab -e
0 3 * * * /home/adminppi/update_unifi_cert.sh >> /var/log/update_unifi_cert.log 2>&1
```