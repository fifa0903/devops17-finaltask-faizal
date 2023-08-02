# terraform (for creating VM)

kita akan melakukan tugas akhir pada OS ubuntu dan akan dimulai dengan terraform untuk membuat vm 

dengan spesifikasi
1. appserver (20 Gb Storage, 2 CPU, 2 Gb Ram)
2. gateway (20 Gb Storage, 2 CPU, 1 Gb Ram)
3. monitoring (20 Gb Storage, 2 CPU, 2 Gb Ram)

buat file main.tf

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/ce8b18ed-77dd-4c65-8037-bb1517d6862a)

kemudian jalankan terraform init, terraform validate, dan terraform apply

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/240e27f7-de2b-4047-bb1d-accda79216ea)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/28cf6748-1762-4903-851c-3499d582f590)

kemudian lakukan hal yang sama untuk gateway dan monitoring

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/eb138845-9f08-4e51-97b6-ecad30915bbb)

![Screenshot from 2023-07-29 09-55-16](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/d35e938f-504f-4c49-8bb1-c31833d9bc1c)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/6153ad8e-60b7-4280-a70e-4ad8dfe43169)

# ansible (for installation inside VM)

buat file ansible.cfg

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/8646410f-600d-422b-81f2-2b2cb2c39077)

setup Inventory

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/fc2f7786-9b2e-4eb0-80da-04f5de4b270f)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/e8bffe97-de6f-48ae-9b65-e82fc88802f6)

### Docker

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/0b392e24-e370-487b-9664-001244966fd2)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/d72c97fc-bae1-4385-8f3c-3b1c9a0d334a)

### node exporter

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/e7e9952c-8441-4e20-ae48-450b795a14cd)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/56d52723-2ab8-4c9c-8312-ad7f0b506c40)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/dcadd138-8540-41db-abfb-1ecd1490f6cb)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/e17d1c0f-5660-433e-8510-9f4688bee627)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/1db80ab1-9cde-4562-82da-23212e4569d7)

### disable password login

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/ac4816cb-2adf-4147-ae1c-c1ce486a8160)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/bd7cb23b-b076-485a-8c98-3d645bdda4c2)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/f39f67d5-a166-47c8-8d51-43adb1cecf94)

### docker
```
- become: true
  gather_facts: false
  hosts: all
  vars:
    - username: nobody1305
  tasks:
    - name: "Updating apt module"
      apt:
        update_cache: true
    - name: "change apt source.list"
      replace:
        path: /etc/apt/sources.list
        regexp: "http://mirrors.idcloudhost.com/ubuntu"
        replace: "http://archive.ubuntu.com/ubuntu"
        backup: yes
    - name: "Install ca-cert, curl, gnupg"
      apt:
        name:
          - ca-certificates
          - curl
          - gnupg
    - name: "Install GPG Key"
      apt_key:
        url: "https://download.docker.com/linux/ubuntu/gpg"
    - name: "install docker repository"
      apt_repository:
        repo: "deb https://download.docker.com/linux/ubuntu focal stable"
    - name: "install docker enginge"
      apt:
        update_cache: true
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-buildx-plugin
          - docker-compose-plugin
    - name: "Install python dependencies"
      apt:
        name: python3-pip
        state: latest
        update_cache: true
      become: true
    - name: "install docker sdk for pyhton"
      pip:
        name: docker
    - name: "docker grup add user"
      shell: sudo usermod -aG docker {{ username }}
```

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
    - name: "create rproxy nodeapp"
      copy:
        dest: "/etc/nginx/sites-enabled/nodeapp.conf"
        content: |
          server {
              server_name nodeapp.faizal.studentdumbways.my.id;

              location / {
                       proxy_pass http://103.31.38.86:9100;
              }
          }
    - name: "create rproxy nodegate"
      copy:
        dest: "/etc/nginx/sites-enabled/nodegate.conf"
        content: |
          server {
              server_name nodegate.faizal.studentdumbways.my.id;

              location / {
                       proxy_pass http://103.226.138.6:9100;
              }
          }
    - name: "create rproxy nodemonit"
      copy:
        dest: "/etc/nginx/sites-enabled/nodemonit.conf"
        content: |
          server {
              server_name nodemonit.faizal.studentdumbways.my.id;

              location / {
                       proxy_pass http://116.193.190.5:9100;
              }
          }

    - name: "reload nginx"
      service:
        name: nginx
        state: reloaded

```
![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/1a727a8e-1efb-41f7-a0d7-7067e5e60717)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/c6a3bf58-3381-4940-808b-9f0a5ac4a0f4)

### monitoring


```
- become: true
  gather_facts: true
  hosts: monitoring
  tasks:

    - name: "create prometheus volume"
      ansible.builtin.file:
        path: /etc/prometheus
        state: directory
    - name: "creating prometheus.yml"
      copy:
        dest: "/etc/prometheus/prometheus.yaml"
        content: |
          global:
           scrape_interval: 10s
          scrape_configs:
           - job_name: 'prometheus_metrics'
             scrape_interval: 5s
             static_configs:
               - targets: ['prometheus.faizal.studentdumbways.my.id']
           - job_name: 'node_exporter_metrics'
             scrape_interval: 5s
             static_configs:
               - targets: ['nodeapp.faizal.studentdumbways.my.id','nodegate.faizal.studentdumbways.my.id','nodemonit.faizal.studentdumbways.my.id']
    - name: "create volume grafana"
      ansible.builtin.file:
        path: /grafana
        state: directory
    - name: "create volume jenkins"
      ansible.builtin.file:
        path: /jenkins
        state: directory
    - name: "pull prometheus"
      community.docker.docker_container:
        name: prometheus
        image: prom/prometheus
        ports:
          -  9090:9090
        volumes:
          - /etc/prometheus/prometheus.yaml:/etc/prometheus/prometheus.yaml
        restart_policy: unless-stopped
        command:
          - --config.file=/etc/prometheus/prometheus.yaml
    - name: "pull grafana"
      community.docker.docker_container:
        name: grafana
        image: grafana/grafana
        ports:
          - 3000:3000
        volumes:
          - ~/grafana:/var/lib/grafana
        user: root
        restart_policy: unless-stopped
    - name: "pull jenkins"
      community.docker.docker_container:
        name: jenkins
        image: jenkins/jenkins
        ports:
          - 8080:8080
          - 50000:50000
        volumes:
          - ~/jenkins:/var/jenkins_home
        user: root
        privileged: true
        restart_policy: unless-stopped
```

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/08e7b0c7-c125-43b6-b941-73d735e02788)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/94ea39b8-60e6-48ef-8d3c-3b860435a633)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/bea734f2-4b9c-4bc5-b3c1-565bb87aa8b8)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/27575623-8c9f-4cf7-a81e-0dd8e5a5a2cc)



![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/dfd077bc-0e30-4763-a6d7-aa4f2a4839dc)

