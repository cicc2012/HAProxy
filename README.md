# HAProxy

---

##  What is HAProxy?

**HAProxy** (High Availability Proxy) is an open-source, high-performance **TCP/HTTP load balancer and proxy server**. It's widely used for distributing web traffic across multiple backend servers, ensuring:
- **Load balancing** (round-robin, least connections, etc.)
- **High availability**
- **Health checks**
- **SSL termination**
- **Sticky sessions**, and more.

It's lightweight, super fast, and battle-tested — often used by big websites and cloud services.

---

##  Can You Run the Load Balancer and Web1 on the Same VM?

**Yes, absolutely!** It’s not ideal for real-world deployments, but it’s totally fine for **learning and testing**.

### Here's how it works:
If you're running both HAProxy (load balancer) and Apache (web server) on the same VM, you'll just treat that Apache as one of the backends.

Let’s say:

- VM1 has HAProxy and Apache running
- VM2 has Apache only

Your HAProxy config on **VM1** could look like this:

```haproxy
frontend http_front
    bind *:80
    default_backend apache_backend

backend apache_backend
    balance roundrobin
    server local_apache 127.0.0.1:8080 check
    server web2 192.168.56.12:80 check
```

But to avoid port conflicts, you’ll need to **run Apache on a different port** (like `8080`) on the same VM:

### Change Apache port (on VM1):
Edit the ports config:
```bash
sudo vim /etc/apache2/ports.conf
```

Change:
```apache
Listen 8080
```

Also, update the virtual host:
```bash
sudo vim /etc/apache2/sites-enabled/000-default.conf
```

Change:
```apache
<VirtualHost *:8080>
```

Then restart Apache:
```bash
sudo systemctl restart apache2
```

Now you have:
- Apache serving at `127.0.0.1:8080`
- HAProxy listening on `0.0.0.0:80` and forwarding to Apache (local + remote)

Test by curling `http://localhost` from the LB VM, and you should see responses from both servers alternating.
