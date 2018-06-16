# Install and Configure Nginx on CentOS 7

**Install Nginx**
```sh
sudo yum install -y nginx
```

**Enable and Start Nginx Service**
```sh
sudo systemctl enable nginx
sudo systemctl start nginx
```

**Configure Firewall for Nginx**
```sh
// default HTTP service on port 80:
sudo firewall-cmd --permanent --zone=public --add-service=http

// or other ports like 8080:
sudo firewall-cmd --permanent --zone=public --add-port=8080/tcp

// Reload firewalld
sudo firewall-cmd --reload
// Check
sudo firewall-cmd --list-all
```

**Check if Nginx Works**
```sh
curl localhost
```

**Use Customized Server Root of Nginx**
- File cấu hình:
    - `/etc/nginx/nginx.conf`
- Nginx User
    - Nginx được chạy với `user` được định nghĩa trong `/etc/nginx/nginx.conf`
        ```sh
        # User mặc định là nginx
        ...
        user nginx;
        ...
        ```
    - Cấu hình thư mục gốc mong muốn
        ```sh
        server {
            ...
            #root         /usr/share/nginx/html;
            root         /home/xxx/html;
            ...
        }
        ```
    - Thư mục mặc định là `/usr/share/nginx/html`
    - Tất cả thư mục (bao gồm thư mục gốc) trong `/home/xxx/html` phải có quyền `rx` cho Nginx user nếu không sẽ bị lỗi 403 forbidden:
        ```sh
        sudo cat /var/log/nginx/error.log

        // Output:
        2017/09/26 12:50:41 [error] 20491#0: *1 "/home/xxx/html/index.html" is forbidden (13: Permission denied), client: ::1, server: _, request: "GET / HTTP/1.1", host: "localhost"

        # Check if other user(nginx) has 'rx' on /home/xxx
        ls /home -la
        drwx------.  3 xxx  xxx   95 Jun  26 12:50 xxx

        # Add 'rx' for other(user nginx):
        sudo chmod o+rx -R /home/xx

        # Or Add Nginx to xxx group
        usermod -a -G xxx nginx
        chmod g+rx /home/xxx
        ``` 
    - Cấu hình SELinux
        - Xem [Fix 502 Error of Nginx on CentOS 7](https://github.com/noname1908/Notes/blob/master/Nginx/configure-selinux-to-fix-502-error-of-nginx-on-centos.md)