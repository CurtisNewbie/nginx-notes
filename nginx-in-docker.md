# Nginx In Docker

In docker-compose.yml file, we create the Nginx container using image `nginx:alpine`. To configure it, we use volumes (mounting) as below. Typically, the `nginx.conf` file is mounted, so that we can provide our custom configuration. 

If we build the image on our own, we can use `COPY` command to copy the `nginx.conf` file into the container. We may also mount the folder that contains the frontend code, and configure it in `nginx.conf`

```yml
version: "3.9"
services:
  nginx:
    image: nginx:alpine
    volumes:
      - /home/zhuangyongj/services/nginx/nginx.conf:/etc/nginx/nginx.conf
      - /home/zhuangyongj/services/nginx/cert:/etc/nginx/cert
      - /home/zhuangyongj/services/nginx/html:/usr/share/nginx/html/ 
    ports:
      - "0.0.0.0:8001:8001"
      - "0.0.0.0:8002:8002"
    networks:
      - backend
    restart: on-failure 

    # other services ...

networks:
  backend:
```

In our `nginx.conf` file, we use the mounted files and folder directly.

- `/usr/share/nginx/html/*`
    - is where the static files are served
- `/etc/nginx/cert/*`
    - is where the ssl certificate and keys are stored
- `proxy_pass http://yourBackend:8082`
    - is the address of the backend server 

```conf

http {
    server {

        listen 8002 ssl;

		ssl_certificate /etc/nginx/cert/curtisnewbie.com.crt;
		ssl_certificate_key /etc/nginx/cert/curtisnewbie.com.key;

        location / {
            index index.html; 

            # where the static files are served            
            root /usr/share/nginx/html/file-service-web;
        }

        location /api/ {
            proxy_redirect off;

            proxy_pass http://file-service:8082;
            proxy_set_header Host $http_host;
        }
    }
}
```

To generate a self-signed certificate, and use it as the example above:

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt
```



