# erosion-deploy-core

Deploy Erosion web portal.

# Install

```shell
#sudo apt install git -y
sudo mkdir -p /opt/erosion/portal
cd /opt/erosion/portal
python3 -m venv venv
source venv/bin/activate
pip install pip --upgrade
pip install -r requirements.txt
```

# Run

```shell
ansible-playbook -i hosts.yml deploy-portal.yml
```



Pedir certificado:

apt install podman newidmap slirp4netns

Error: command required for rootless mode with multiple IDs: exec: "newuidmap": executable file not found in $PATH

El modo rootless en podman usa user namespaces para crear un entorno aislado. Para mapear los UID y GID correctamente, podman necesita estos dos binarios:

    newuidmap

    newgidmap

Estos generalmente vienen con el paquete uidmap.



Error: could not find slirp4netns, the network namespace can't be configured: exec: "slirp4netns": executable file not found in $PATH

Es un programa que permite crear redes virtuales para contenedores sin necesidad de privilegios de root, usando SLIRP (una emulación de red).


mkdir -p /opt/certbot/etc
mkdir -p /opt/certbot/var
chown -R ceon:ceon /opt/certbot
su - ceon

podman run -it --rm --name certbot \
    -v "/opt/certbot/etc:/etc/letsencrypt" \
    -v "/opt/certbot/var:/var/lib/letsencrypt" \
    -p 8094:80 \
    docker.io/certbot/certbot certonly --standalone \
    -d erosion.es -m cctpidmdlcdtc@gmail.com \
    --http-01-port=80 --agree-tos -n -v --pre-hook "/bin/sleep 15"




# Probar turnserver

```python
import hmac
import hashlib
import base64
import time

# Config
secret = b"pleasedontihaveacat"
username_expiry = int(time.time()) + 600  # Expira en 10 minutos
username = str(username_expiry)
realm = "erosion.es"

# HMAC-SHA1
key = hmac.new(secret, username.encode('utf-8'), hashlib.sha1).digest()
password = base64.b64encode(key).decode('utf-8')

print(f"username: {username}")
print(f"password: {password}")
print(f"realm: {realm}")
```

```
username: 1748542657
password: 4baxD6T+WY9s6FaPRtz/oMjjeHI=
realm: erosion.es
```

```shell
turnutils_uclient -v erosion.es -e 127.0.0.1 -y -u 1748542657 -w "4baxD6T+WY9s6FaPRtz/oMjjeHI=" -r erosion.es
```



