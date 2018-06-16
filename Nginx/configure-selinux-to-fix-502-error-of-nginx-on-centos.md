# Configure SELinux to Fix 502 Error of Nginx on CentOS 7

**Problem**

- Lỗi 502 xảy ra khi truy cập Nginx trên CentOS 7.

**Details**

- Phiên bản CentOS 7 1708
- Nginx được cài đặt bằng `yum`
- Lỗi 502 xảy ra khi truy cập server nodejs
    - Start server nodejs http://127.0.0.1:3333
    - Cấu hình ứng dụng Nodejs `/etc/nginx/conf.d/blog.conf`
        ```sh
        upstream blog_server {
            server 127.0.0.1:3333 fail_timeout=0;
        }
        server {
            listen 80;
            listen [::]:80;
            server_name [none-www url];
            return 301 [www url]$request_uri;
        }
        server {
            listen 80;
            server_name [www url];
            root  [project folder];

            # Set path for access_logs
            #access_log [project folder]/access.log combined;

            # Set path for error logs
            #error_log [project folder]/error.log notice; 

            # If set to on, Nginx will issue log messages for every operation
            # performed by the rewrite engine at the notice error level
            # Default value off
            rewrite_log on;

            client_max_body_size 5M;
            
            location / {
            # Header settings for application behind proxy
            proxy_set_header Host $host;
            # proxy_set_header X-NginX-Proxy true;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";

            # Proxy pass settings
            proxy_pass http://blog_server;

            # Proxy redirect settings
            proxy_redirect off;

            # HTTP version settings
            proxy_http_version 1.1;

            # Response buffering from proxied server default 1024m
            proxy_max_temp_file_size 0;

            # Proxy cache bypass define conditions under the response will not be taken from cache
            proxy_cache_bypass $http_upgrade;
            }
        }

        ```

    - Khởi động lại Nginx
        ```sh
        sudo systemctl restart nginx
        ```

    - Test
        - Truy cập trình duyệt với đường dẫn đã cấu hình và nhận được lỗi 502.

- Dịch vụ Nginx chạy với user mặc định `nginx`

**Root Cause**
- SELinux Policies.

**How to Fix**
- Cài đặt công cụ audit2allow
    ```sh
    yum provides audit2allow
    // Output: 
    // policycoreutils-python-2.5-17.1.el7.x86_64 : SELinux policy core python utilities

    sudo yum install -y policycoreutils-python
    ```

- Cài đặt và áp dụng các quy tắc chưa được chấp nhận cho Nginx.
    - Cách 1: áp dụng các quy tắc chưa được chấp nhận có trong `audit.log`
        1. Check audit.log
            ```sh
            sudo cat /var/log/audit/audit.log | grep nginx | grep denied

            // You may find messages like:
            type=AVC msg=audit(1506398841.964:345): avc:  denied  { getattr } for  pid=20510 comm="nginx" path="/var/www/html/index.html" dev="dm-0" ino=635249 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:var_t:s0 tclass=file
            ```

        2. Tạo mới module
            ```sh
            sudo cat /var/log/audit/audit.log | grep nginx | grep denied | sudo audit2allow -M nginx
            ```

        3. Áp dụng policy module:
            ```sh
            sudo semodule -i nginx.pp
            ```

        4. Kiểm tra: Truy cập trình duyệt với đường dẫn đã cấu hình đã hiển thị được trang web.

        5. Nếu vẫn lỗi thì lặp lại các bước 1 - 4 đến khi không còn nhận được lỗi 502.
            - Xem lỗi mới sau khi chạy lệnh `sudo cat /var/log/audit/audit.log | grep nginx | grep denied`
            - Có thể phải làm lại **nhiều lần** vì có nhiều SELinux Policies khác nhau.

    - Cách 2: tùy chỉnh file `nginx.te`
        1. Tạo SELinux policy module tùy chỉnh
            ```sh
            grep nginx /var/log/audit/audit.log | audit2allow -m nginx > nginx.te
            cat nginx.te


            module nginx 1.0;

            require {
                type var_run_t;
                type user_home_dir_t;
                type httpd_log_t;
                type httpd_t;
                type user_home_t;
                type httpd_sys_content_t;
                type initrc_t;
                type http_cache_port_t;
                class sock_file write;
                class unix_stream_socket connectto;
                class dir { search getattr };
                class file { read write setattr };
                class tcp_socket name_connect;
            }

            #============= httpd_t ==============

            #!!!! This avc is allowed in the current policy
            allow httpd_t http_cache_port_t:tcp_socket name_connect;
            allow httpd_t httpd_log_t:file setattr;
            allow httpd_t httpd_sys_content_t:sock_file write;
            allow httpd_t initrc_t:unix_stream_socket connectto;

            #!!!! This avc is allowed in the current policy
            allow httpd_t user_home_dir_t:dir search;

            #!!!! This avc is allowed in the current policy
            allow httpd_t user_home_t:dir { search getattr };
            allow httpd_t user_home_t:sock_file write;
            allow httpd_t var_run_t:file { read write };
            ```
        2. Chạy lệnh để tạo policy module
            ```sh
            grep nginx /var/log/audit/audit.log | audit2allow -M nginx
            ```
        3. Áp dụng policy module
            ```sh
            semodule -i nginx.pp
            ```

### References

* [8.3.8. Allowing Access: audit2allow](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/sect-Security-Enhanced_Linux-Fixing_Problems-Allowing_Access_audit2allow.html)
* [centos7 中关于 nginx 的权限问题](https://www.v2ex.com/t/171804)
* [(13: Permission denied) while connecting to upstream:[nginx]](https://stackoverflow.com/questions/23948527/13-permission-denied-while-connecting-to-upstreamnginx)