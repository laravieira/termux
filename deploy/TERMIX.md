# Termux Termix Deploy
How to deploy the Termix dashboard on the environment set by following this repository guides.

### Download termix
```shell
mkdir ~/projects/termix
cd ~/projects/termix
git clone https://github.com/Termix-SSH/Termix .
```

### Build front-end
```shell
npm install --ignore-scripts --force
npm cache clean --force
find public/fonts -name "*.ttf" ! -name "*Regular.ttf" ! -name "*Bold.ttf" ! -name "*Italic.ttf" -delete
NODE_OPTIONS="--max-old-space-size=3072" npm run build
```

### Build back-end
```shell
npm rebuild better-sqlite3 --force
npm run build:backend
```

### Start the back-end
```shell
node dist/backend/backend/starter.js
```

### Configure nginx
Termix run multiple services and host static files, it heavily needs to use a proxy server, as it configures nginx by default, install nginx folloing [this tutorial](/NGINX.md) and add (append), into the `http {}` block, this server configuration:
###### the root properties of all locations must point to your instalation `dist` folder.

`$PREFIX/etc/nginx/nginx.conf`
```
http {
    server {
        listen 8081;
        server_name termix;

        add_header X-Content-Type-Options nosniff always;
        add_header X-XSS-Protection "1; mode=block" always;

        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            root /data/data/com.termux/files/home/projects/termix/dist;
            expires 1y;
            add_header Cache-Control "public, immutable";
            try_files $uri =404;
        }

        location / {
            root /data/data/com.termux/files/home/projects/termix/dist;
            index index.html index.htm;
            try_files $uri $uri/ /index.html;
        }

        location ~* \.map$ {
            return 404;
            access_log off;
            log_not_found off;
        }

        location ~ ^/users/sessions(/.*)?$ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/users(/.*)?$ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/version(/.*)?$ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/releases(/.*)?$ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/alerts(/.*)?$ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/rbac(/.*)?$ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/credentials(/.*)?$ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_connect_timeout 60s;
            proxy_send_timeout 300s;
            proxy_read_timeout 300s;
        }

        location ~ ^/snippets(/.*)?$ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/terminal(/.*)?$ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/database(/.*)?$ {
            client_max_body_size 5G;
            client_body_timeout 300s;

            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_connect_timeout 60s;
            proxy_send_timeout 300s;
            proxy_read_timeout 300s;

            proxy_request_buffering off;
            proxy_buffering off;
        }

        location ~ ^/db(/.*)?$ {
            client_max_body_size 5G;
            client_body_timeout 300s;

            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_connect_timeout 60s;
            proxy_send_timeout 300s;
            proxy_read_timeout 300s;

            proxy_request_buffering off;
            proxy_buffering off;
        }

        location ~ ^/encryption(/.*)?$ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /ssh/quick-connect {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/ssh/opkssh-chooser(/.*)?$ {
            proxy_pass http://127.0.0.1:30001/ssh/opkssh-chooser$1$is_args$args;
            proxy_http_version 1.1;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-Host $http_host;
            proxy_set_header X-Forwarded-Port $server_port;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/ssh/opkssh-callback(/.*)?$ {
            proxy_pass http://127.0.0.1:30001/ssh/opkssh-callback$1$is_args$args;
            proxy_http_version 1.1;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-Host $http_host;
            proxy_set_header X-Forwarded-Port $server_port;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /ssh/ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /ssh/websocket/ {
            proxy_pass http://127.0.0.1:30002/;
            proxy_http_version 1.1;

            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-Host $http_host;
            proxy_set_header X-Forwarded-Port $server_port;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;

            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_read_timeout 86400s;
            proxy_send_timeout 86400s;
            proxy_connect_timeout 10s;

            proxy_buffering off;
            proxy_request_buffering off;

            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
        }

        location /ssh/tunnel/ {
            proxy_pass http://127.0.0.1:30003;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /ssh/file_manager/recent {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /ssh/file_manager/pinned {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /ssh/file_manager/shortcuts {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /ssh/file_manager/sudo-password {
            proxy_pass http://127.0.0.1:30004;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /ssh/file_manager/ssh/ {
            client_max_body_size 5G;
            client_body_timeout 300s;

            add_header Cache-Control "no-store, no-cache, must-revalidate" always;

            proxy_pass http://127.0.0.1:30004;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_connect_timeout 60s;
            proxy_send_timeout 300s;
            proxy_read_timeout 300s;

            proxy_request_buffering off;
            proxy_buffering off;
        }

        location ~ ^/network-topology(/.*)?$ {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /health {
            proxy_pass http://127.0.0.1:30001;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/status(/.*)?$ {
            proxy_pass http://127.0.0.1:30005;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/metrics(/.*)?$ {
            proxy_pass http://127.0.0.1:30005;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }

        location ~ ^/uptime(/.*)?$ {
            proxy_pass http://127.0.0.1:30006;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/activity(/.*)?$ {
            proxy_pass http://127.0.0.1:30006;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ~ ^/dashboard/preferences(/.*)?$ {
            proxy_pass http://127.0.0.1:30006;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location ^~ /docker/console/ {
            proxy_pass http://127.0.0.1:30008/;
            proxy_http_version 1.1;

            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;

            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_read_timeout 86400s;
            proxy_send_timeout 86400s;
            proxy_connect_timeout 10s;

            proxy_buffering off;
            proxy_request_buffering off;

            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
        }

        location ~ ^/docker(/.*)?$ {
            proxy_pass http://127.0.0.1:30007;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_connect_timeout 60s;
            proxy_send_timeout 300s;
            proxy_read_timeout 300s;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /data/data/com.termux/files/home/projects/termix/dist;
        }
    }
}
```

### Configure as a service
```shell
mkdir -p $PREFIX/var/service/termix/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/termix/log/run
touch $PREFIX/var/service/termix/run
chmod +x $PREFIX/var/service/termix/run
```
`$PREFIX/var/service/termix/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd ~/projects/termix
exec 2>&1
node dist/backend/backend/starter.js
```

### Enable the service and start it
```shell
sv enable termix
sv up termix
```