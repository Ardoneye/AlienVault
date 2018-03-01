#!/bin/sh
# Adjust cert lifetime (v5.5.0) to 1460 days
sed -i /usr/sbin/openvas-mkcert -e "s/SRVCERT_LIFETIME=\"365\"/SRVCERT_LIFETIME=\"1460\"/"
sed -i /usr/sbin/openvas-mkcert-client -e "s/default_days.*=.*/default_days = 1460/"
sed -i /usr/sbin/openvas-mkcert-client -e "s/^DFL_CERT_LIFETIME=.*$/DFL_CERT_LIFETIME=\$\{x\:\-1460\}/"

# Recreate certs
openvas-mkcert -f -q

openvas-mkcert-client -n -i

# Install scanner key
scannerid=$(openvasmd --get-scanners | grep Default | head -n 1 | cut -d" " -f1)

openvasmd --modify-scanner $scannerid --scanner-ca-pub /var/lib/openvas/CA/cacert.pem --scanner-key-pub /var/lib/openvas/CA/clientcert.pem --scanner-key-priv /var/lib/openvas/private/CA/clientkey.pem

# Restart services
service openvas-scanner restart
service openvas-manager restart

# After the scanner is reloaded, rebuild the NVT cache

# Wait until the scanner is done reloading NVT cache
while [ $(ps aux | grep openvasssd | grep Reloaded | wc -l) -ne 0 ]
do
		echo "Waiting 3 secs for the OpenVas scanner to reload..."
		sleep 3
		
done

openvasmd --rebuild --verbose --progress
