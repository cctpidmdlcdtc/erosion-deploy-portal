# El flujo seguro

Cliente malicioso → HAProxy → Nginx → Python
    |               |         |        |
    |               |         |        ├─ Ve: X-Verified-IP: 203.0.113.195 ✅
    |               |         ├─ Recibe: X-Real-Client-IP: 203.0.113.195 ✅  
    |               ├─ Ve IP real: 203.0.113.195 ✅
    ├─ Envía headers falsos: X-Real-IP: 192.168.1.100 ❌

HAProxy actúa como "firewall de headers":


Cliente malicioso envía:

http

GET / HTTP/1.1
Host: ejemplo.com
X-Real-IP: 192.168.1.50
X-Forwarded-For: 10.0.0.100
X-Client-IP: 172.16.0.1

HAProxy procesa y envía a Nginx:

http

GET / HTTP/1.1
Host: ejemplo.com
X-Real-Client-IP: 203.0.113.195    # <- IP real del socket
# Todos los headers falsos fueron eliminados

Python recibe de Nginx:

http

GET / HTTP/1.1
Host: ejemplo.com  
X-Verified-Client-IP: 203.0.113.195  # <- 100% confiable



Configuración completa y segura:
HAProxy

```shell
frontend web_frontend
    bind *:80
    
    # Blacklist de headers peligrosos
    http-request del-header X-Forwarded-For
    http-request del-header X-Real-IP
    http-request del-header X-Client-IP
    http-request del-header X-Forwarded
    http-request del-header CF-Connecting-IP
    http-request del-header True-Client-IP
    http-request del-header X-Originating-IP
    
    # Solo la verdad
    http-request add-header X-Real-Client-IP %[src]
    
    default_backend nginx_backend
```

Nginx

```shell
server {
    listen 8080;
    
    location / {
        proxy_pass http://python_app;
        proxy_set_header X-Verified-Client-IP $http_x_real_client_ip;
        
        # Por seguridad extra, no pasar otros headers de IP
        proxy_set_header X-Forwarded-For "";
        proxy_set_header X-Real-IP "";
    }
}
```

Python

```python
def get_client_ip():
    # 100% confiable - viene de HAProxy que vio el socket real
    return request.headers.get('X-Verified-Client-IP')

def is_whitelisted():
    client_ip = get_client_ip()
    # Esta IP es completamente confiable
    return client_ip in WHITELIST
```


¿Hay alguna forma de burlar esto?

No, a menos que:

    El atacante comprometa tu HAProxy ❌
    El atacante tenga acceso a tu red interna ❌
    Haya un bug en HAProxy ❌
    El atacante secuestre la conexión TCP (extremadamente difícil) ❌

En condiciones normales: IMPOSIBLE de spoofear ✅








Implementar en nftables el filtrado por ips, a partir de una whitelist




table inet filter {
    # Set dinámico para IPs permitidas por Python
    set coturn_dynamic_ips {
        type ipv4_addr
        flags dynamic,timeout
        timeout 1h
        # Tamaño máximo del set (ajustar según necesidades)
        size 1000
    }

    chain input {
        type filter hook input priority 0; policy drop;
        
        # Permitir tráfico de loopback
        iif lo accept
        
        # Permitir tráfico establecido y relacionado
        ct state established,related accept
        
        ip saddr @coturn_dynamic_ips tcp dport $COTURN_STUN_PORT accept
