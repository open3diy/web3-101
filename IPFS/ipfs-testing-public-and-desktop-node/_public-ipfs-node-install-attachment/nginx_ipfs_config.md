# Web3 - 101 - IPFS - Probando un Nodo Público y de Escritorio - Instalación de IPFS con docker en un nodo público

## Pasos

### Crear configuración de Ngins para IPFS

Crear archivo de configuración en `/etc/appserver/nginx/conf.d/ipfs.conf`:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name *.ipfs.web3-101-ipfs.open3diy.org *.ipns.web3-101-ipfs.open3diy.org;
    
    ssl_certificate /etc/letsencrypt/live/web3-101-ipfs.open3diy.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/web3-101-ipfs.open3diy.org/privkey.pem;

    include conf.d/gateway-ipfs/denylist.conf;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 1800s;
        proxy_pass http://ipfs-host:8080;
    }

}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name web3-101-ipfs.open3diy.org;
    
    ssl_certificate /etc/letsencrypt/live/web3-101-ipfs.open3diy.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/web3-101-ipfs.open3diy.org/privkey.pem;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 1800s;
        proxy_pass http://ipfs-host:8080;
    }
}
```

> Recuerda revisar donde aparezca `web3-101-ipfs.open3diy.org` indicando el dominio que corresponda a tú caso.

Verificar que la configuración es válida en el contenedo:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -t
```

- **salida:**

  ```plaintext
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful
  ```

Recarga configuracion nginx sin reiniciar el contenedor:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -s reload
```

Revisar logs:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs reverse-proxy
```

> Misma salida anterior con `reverse-proxy  | /docker-entrypoint.sh: Configuration complete; ready for start up`.

Prueba con las cabeceras Http necesarias, para la petición como ruta:

```bash
curl -v "https://web3-101-ipfs.open3diy.org/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m"
```

- **salida:**

  ```plaintext  
  * Host web3-101-ipfs.open3diy.org:443 was resolved.
  * IPv6: (none)
  * IPv4: 57.129.131.125
  *   Trying 57.129.131.125:443...
  * Connected to web3-101-ipfs.open3diy.org (57.129.131.125) port 443
  * ALPN: curl offers h2,http/1.1
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  *  CAfile: /usr/lib/ssl/cert.pem
  *  CApath: /usr/lib/ssl/certs
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
  * TLSv1.3 (IN), TLS handshake, Certificate (11):
  * TLSv1.3 (IN), TLS handshake, CERT verify (15):
  * TLSv1.3 (IN), TLS handshake, Finished (20):
  * TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.3 (OUT), TLS handshake, Finished (20):
  * SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519 / id-ecPublicKey
  * ALPN: server accepted http/1.1
  * Server certificate:
  *  subject: CN=web3-101-ipfs.open3diy.org
  *  start date: Feb 27 16:18:26 2025 GMT
  *  expire date: May 28 16:18:25 2025 GMT
  *  subjectAltName: host "web3-101-ipfs.open3diy.org" matched cert's "web3-101-ipfs.open3diy.org"
  *  issuer: C=US; O=Let's Encrypt; CN=E6
  *  SSL certificate verify ok.
  *   Certificate level 0: Public key type EC/prime256v1 (256/128 Bits/secBits), signed using ecdsa-with-SHA384
  *   Certificate level 1: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using sha256WithRSAEncryption
  *   Certificate level 2: Public key type RSA (4096/152 Bits/secBits), signed using sha256WithRSAEncryption
  * using HTTP/1.x
  > GET /ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m HTTP/1.1
  > Host: web3-101-ipfs.open3diy.org
  > User-Agent: curl/8.5.0
  > Accept: */*
  > 
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * old SSL session ID is stale, removing
  < HTTP/1.1 301 Moved Permanently
  < Server: nginx/1.27.4
  < Date: Sun, 02 Mar 2025 18:18:58 GMT
  < Content-Type: text/html; charset=utf-8
  < Content-Length: 135
  < Connection: keep-alive
  < Access-Control-Allow-Headers: Content-Type
  < Access-Control-Allow-Headers: Range
  < Access-Control-Allow-Headers: User-Agent
  < Access-Control-Allow-Headers: X-Requested-With
  < Access-Control-Allow-Methods: GET
  < Access-Control-Allow-Methods: HEAD
  < Access-Control-Allow-Methods: OPTIONS
  < Access-Control-Allow-Origin: *
  < Access-Control-Expose-Headers: Content-Length
  < Access-Control-Expose-Headers: Content-Range
  < Access-Control-Expose-Headers: X-Chunked-Output
  < Access-Control-Expose-Headers: X-Ipfs-Path
  < Access-Control-Expose-Headers: X-Ipfs-Roots
  < Access-Control-Expose-Headers: X-Stream-Output
  < Location: https://bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org/
  < Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  < 
  <a href="https://bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org/">Moved Permanently</a>.

  * Connection #0 to host web3-101-ipfs.open3diy.org left intact
  ```

Prueba con las cabeceras Http necesarias, para la petición como subdominio:

```bash
curl -v "https://bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org"
```

- **salida:**

  ```plaintext  
  * Host bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org:443 was resolved.
  * IPv6: (none)
  * IPv4: 57.129.131.125
  *   Trying 57.129.131.125:443...
  * Connected to bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org (57.129.131.125) port 443
  * ALPN: curl offers h2,http/1.1
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  *  CAfile: /usr/lib/ssl/cert.pem
  *  CApath: /usr/lib/ssl/certs
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
  * TLSv1.3 (IN), TLS handshake, Certificate (11):
  * TLSv1.3 (IN), TLS handshake, CERT verify (15):
  * TLSv1.3 (IN), TLS handshake, Finished (20):
  * TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.3 (OUT), TLS handshake, Finished (20):
  * SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519 / id-ecPublicKey
  * ALPN: server accepted http/1.1
  * Server certificate:
  *  subject: CN=web3-101-ipfs.open3diy.org
  *  start date: Feb 27 16:18:26 2025 GMT
  *  expire date: May 28 16:18:25 2025 GMT
  *  subjectAltName: host "bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org" matched cert's "*.ipfs.web3-101-ipfs.open3diy.org"
  *  issuer: C=US; O=Let's Encrypt; CN=E6
  *  SSL certificate verify ok.
  *   Certificate level 0: Public key type EC/prime256v1 (256/128 Bits/secBits), signed using ecdsa-with-SHA384
  *   Certificate level 1: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using sha256WithRSAEncryption
  *   Certificate level 2: Public key type RSA (4096/152 Bits/secBits), signed using sha256WithRSAEncryption
  * using HTTP/1.x
  > GET / HTTP/1.1
  > Host: bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org
  > User-Agent: curl/8.5.0
  > Accept: */*
  > 
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * old SSL session ID is stale, removing
  < HTTP/1.1 200 OK
  < Server: nginx/1.27.4
  < Date: Sun, 02 Mar 2025 18:22:17 GMT
  < Content-Type: text/plain; charset=utf-8
  < Content-Length: 32
  < Connection: keep-alive
  < Accept-Ranges: bytes
  < Access-Control-Allow-Headers: Content-Type
  < Access-Control-Allow-Headers: Range
  < Access-Control-Allow-Headers: User-Agent
  < Access-Control-Allow-Headers: X-Requested-With
  < Access-Control-Allow-Methods: GET
  < Access-Control-Allow-Methods: HEAD
  < Access-Control-Allow-Methods: OPTIONS
  < Access-Control-Allow-Origin: *
  < Access-Control-Expose-Headers: Content-Length
  < Access-Control-Expose-Headers: Content-Range
  < Access-Control-Expose-Headers: X-Chunked-Output
  < Access-Control-Expose-Headers: X-Ipfs-Path
  < Access-Control-Expose-Headers: X-Ipfs-Roots
  < Access-Control-Expose-Headers: X-Stream-Output
  < Cache-Control: public, max-age=29030400, immutable
  < Etag: "bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m"
  < X-Ipfs-Path: /ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m/
  < X-Ipfs-Roots: bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m
  < Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  < 
  Hello from IPFS Gateway Checker
  * Connection #0 to host bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org left intact
  ```

Prueba el servicio de gateway con la URL <https://web3-101-ipfs.open3diy.org/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m> y como subdominio <https://bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org>.

> El valor `bafybeiack3rhfn25rzsvi7s7f3m37zvrwprthspsoitekib3orqxa7ycpi` es el CID que creamos anteriormente o uno cualquiera que podamos explorar.

Luego prueba IPNS <https://web3-101-ipfs.open3diy.org/ipns/k51qzi5uqu5di56caajjiel546q92pme0hgnh4gofey4tbwlfdr64ur7vu9s9t>.

Revisar los logs de acceso:

```bash
less +G /var/log/nginx/access.log
```
