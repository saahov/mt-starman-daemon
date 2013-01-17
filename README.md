# Run Movable Type with nginx and PSGI on Debian

## 1. Debian

 1. Install cpanminus:
```sh
curl -L http://cpanmin.us | perl - --sudo App::cpanminus`
```

 2. Install Perl modules:
```sh
cpanm Task::Plack XMLRPC::Transport::HTTP::Plack Starman
```

 3. Place in `/etc/init.d` `mt-starman`:
```sh
wget -P /etc/init.d https://raw.github.com/saahov/mt-starman-daemon/master/debian/mt-starman
```

 4. Change permissions:
```sh
chmod +x /etc/init.d/mt-starman
```

 5. Change variables (`DIR`, `SCRIPT`, `USER`, `GROUP`, `WORKERS`) in `/etc/init.d/mt-starman`.

 6. Add `mt-starman` to Debian autostart:
```sh
update-rc.d mt-starman defaults
```

 If you use Debian Squeeze (and above):
```sh
insserv mt-starman
```

## 2. nginx

 1. Add to `nginx.conf` (`http` section):
```nginx
upstream starman {
    server 127.0.0.1:50000;
}
```

 2. Add to nginx site config (`server` section):
```nginx
location /cgi-bin/mt/ {
    proxy_set_header        Host $http_host;
    proxy_set_header        X-Forwarded-Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass      http://starman;
}
```

## 3. Movable Type

 1. Add to `mt-config.cgi`:
```perl
PIDFilePath /var/run/mt.pid
```

 2. Also you can add this in `mt-config.cgi` (for beautiful url's without extensions):
```perl
AdminScript app
CommentScript comments
TrackbackScript trackbacks
SearchScript search
XMLRPCScript xmlrpc
AtomScript atom
UpgradeScript upgrade
```

### Finally

Run this command to start Starman and restart nginx:
```bash
/etc/init.d/mt-starman start && /etc/init.d/nginx restart
```