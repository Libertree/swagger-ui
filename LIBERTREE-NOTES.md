Below are example configurations for web server frontends/proxies.

## Apache

    <VirtualHost *:80>
        ServerName yourtree.org
        ServerAlias yourtree.org
        ErrorLog /var/log/apache2/yourtree.org.errors
        CustomLog /var/log/apache2/yourtree.org.log combined

        <Proxy balancer://libertreecluster>
            BalancerMember http://192.168.0.100:8088
        </Proxy>

        ProxyPreserveHost On

        # Don't proxy requests for Swagger UI
        ProxyPass /api/explore !

        ProxyPass / balancer://libertreecluster/
        ProxyPassReverse / balancer://libertreecluster/

        # Swagger UI:

        Alias /api/explore /misc/git/swagger-ui/dist

        <Directory "/misc/git/swagger-ui/dist">
            Options Indexes FollowSymLinks -ExecCGI
            AllowOverride All
            Order allow,deny
            Allow from all
        </Directory>
    </VirtualHost>

## nginx

    server {
        listen 80;
        server_name  yourtree.org;
        root         /home/libertree/git/libertree-frontend-ramaze/public;

        location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_redirect off;
            proxy_read_timeout 90s;

            client_max_body_size 5M;
            client_body_buffer_size 128K;

            if (!-f $request_filename) {
                proxy_pass http://unicorn_cluster;
                break;
            }
        }

        # Swagger UI
        location /api/explore {
            alias /home/libertree/git/swagger-ui/dist;
        }
    }
