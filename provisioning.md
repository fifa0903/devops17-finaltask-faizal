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
    - name: "reload nginx"
      service:
        name: nginx
        state: reloaded

```
![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/1a727a8e-1efb-41f7-a0d7-7067e5e60717)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/c6a3bf58-3381-4940-808b-9f0a5ac4a0f4)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/dfd077bc-0e30-4763-a6d7-aa4f2a4839dc)

