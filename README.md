# config_ovh

sudo docker run --restart=always -d -p 80:80 -p 443:443 --name=moncozy --volume=/home/cozy/backup:/media -e DOMAIN=trololu.ovh cozy/full

docker exec -it moncozy /bin/bash

git clone https://github.com/certbot/certbot /root/letsencrypt

/etc/init.d/nginx stop

mv /etc/cozy/server.key /etc/cozy/server.key.backup
mv /etc/cozy/server.crt /etc/cozy/server.crt.backup

/root/letsencrypt/letsencrypt-auto certonly \
   --standalone --agree-tos \
   --email gervail.antoine@gmail.com -d trololu.ovh -d www.trololu.ovh \
   --preferred-challenges tls-sni-01
   
ln -s /etc/letsencrypt/live/trololu.ovh/privkey.pem /etc/cozy/server.key
ln -s /etc/letsencrypt/live/trololu.ovh/fullchain.pem /etc/cozy/server.crt

/etc/init.d/nginx start

cat > /root/renew_cert.sh

#!/bin/sh
/etc/init.d/nginx stop
/root/letsencrypt/letsencrypt-auto certonly --standalone -d trololu.ovh -d www.trololu.ovh --preferred-challenges tls-sni-01 --renew-by-default
/etc/init.d/nginx start


chmod u+x /root/renew_cert.sh

crontab -e
0 3 1 * * /root/renew_cert.sh


docker exec moncozy cp -a /var/lib/couchdb/cozy.couch /home/cozy/cozy.couch

