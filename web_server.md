### nginx
```
- become: true
  gather_facts: false
  hosts: gateway
  tasks:
    - name: "install nginx"
      apt:
        name: nginx
        state: present
        update_cache: yes
    - name: "start nginx"
      service:
        name: nginx
        state: started
    - name: "create rproxy frontend"
      copy:
        dest: "/etc/nginx/sites-enabled/frontend.conf"
        content: |
          server {
              server_name faizal.studentdumbways.my.id;

              location / {
                       proxy_pass http://103.31.38.86:3000;
              }
          }
    - name: "create rproxy backend"
      copy:
        dest: "/etc/nginx/sites-enabled/backend.conf"
        content: |
          server {
              server_name api.faizal.studentdumbways.my.id;

              location / {
                       proxy_pass http://103.31.38.86:5000;
              }
          }
    - name: "create rproxy grafana"
      copy:
        dest: "/etc/nginx/sites-enabled/grafana.conf"
        content: |
          server {
              server_name grafana.faizal.studentdumbways.my.id;

              location / {
                       proxy_pass http://116.193.190.5:3000;
              }
          }
    - name: "create rproxy prometheus"
      copy:
        dest: "/etc/nginx/sites-enabled/prometheus.conf"
        content: |
          server {
              server_name prometheus.faizal.studentdumbways.my.id;

              location / {
                       proxy_pass http://116.193.190.5:9090;
              }
          }
    - name: "create rproxy jenkins"
      copy:
        dest: "/etc/nginx/sites-enabled/jenkins.conf"
        content: |
          server {
              server_name jenkins.faizal.studentdumbways.my.id;

              location / {
                       proxy_pass http://116.193.190.5:8080;
              }
          }
    - name: "reload nginx"
      service:
        name: nginx
        state: reloaded

```
![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/e81591e2-42f0-473d-9d45-212942c4d212)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/1a727a8e-1efb-41f7-a0d7-7067e5e60717)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/c6a3bf58-3381-4940-808b-9f0a5ac4a0f4)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/dfd077bc-0e30-4763-a6d7-aa4f2a4839dc)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/a2625a19-0f78-4460-b45d-5a3c70e2f9dd)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/6b2d9eef-e154-4e61-8baa-0da6a6dd5a9f)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/213439d3-b718-487a-bd70-fa563b19e57f)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/4f2ff990-0925-4452-ba93-34a7eafd2314)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/c3926786-c293-4122-ae9b-35480d2b5258)
