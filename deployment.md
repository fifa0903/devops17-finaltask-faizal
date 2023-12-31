# fe-dumbmerch

```
- become: true
  gather_facts: false
  hosts: appserver
  tasks:
    - name: "Ensure repo is up-to-date"
      git:
        repo: https://github.com/fifa0903/fe-dumbmerch
        dest: /home/nobody1305/fe-dumbmerch

    - name: "making dockerfile"
      copy:
        dest: "/home/nobody1305/fe-dumbmerch/Dockerfile"
        content: |
          FROM node:16.20.1-alpine
          WORKDIR /app
          COPY . .
          RUN npm install
          EXPOSE 3000
          CMD ["npm", "start"]
    - name: "making docker compose"
      copy:
        dest: "/home/nobody1305/docker-compose.yml"
        content: |
          version: "3.8"
          services:
            frontend:
              build: ./fe-dumbmerch
              container_name: fe-dumbmerch
              stdin_open: true
              ports:
                - 3000:3000
# if there is error, try this code
#    - name: "Install docker-compose python package"
#      ansible.builtin.pip:
#       name: docker-compose
    - name: "Create and start services"
      community.docker.docker_compose:
        project_src: "/home/nobody1305"
        files:
          - docker-compose.yml

```
![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/b753f6d1-82ff-47ae-839c-4b4a2dbf0ebb)

jalankan postgresql dan masuk untuk membuat database

```
- become: true
  gather_facts: false
  hosts: appserver
  tasks:
    - name: "create postgresql.yml"
      copy:
        dest: "/home/nobody1305/postgresql.yml"
        content: |
          version: '3.8'
          services:
             db:
               image: postgres
               restart: always
               container_name: db-dumbmerch
               expose:
                 - 5432
               ports:
                 - 5432:5432
               volumes:
                 - ~/postgresql:/var/lib/postgresql/data
               environment:
                 - POSTGRES_ROOT_PASSWORD=faizal1305
                 - POSTGRES_USER=nobody1305
                 - POSTGRES_PASSWORD=faizal1305
                 - POSTGRES_DATABASE=dumbmerch
    - name: "Install docker-compose python package"
      ansible.builtin.pip:
        name: docker-compose
    - name: "start postgresql.yml"
      community.docker.docker_compose:
        project_src: "/home/nobody1305"
        files:
          - postgresql.yml
        recreate: smart

```
![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/b71e5e64-6594-49e0-bec7-fd53d5040857)

kemudian setup buat backend 

```
- become: true
  gather_facts: false
  hosts: appserver
  tasks:
#    - name: "Ensure repo is up-to-date"
#      git:
#        repo: https://github.com/fifa0903/be-dumbmerch
#        dest: /home/nobody1305/be-dumbmerch
    - name: "setup .env"
      copy:
        dest: "/home/nobody1305/be-dumbmerch/.env"
        content: |
          #SECRET_KEY=bolehapaaja
          #PATH_FILE=http://localhost:5000/uploads/
          #SERVER_KEY=SB-Mid-server-fJAy6udMPnJCIyFguce8Eot3
          #CLIENT_KEY=SB-Mid-client-YUogx3u74Gq9MTMS
          #EMAIL_SYSTEM=demo.dumbways@gmail.com
          #PASSWORD_SYSTEM=rbgmgzzcmrfdtbpu
            DB_HOST=103.31.38.86
            DB_USER=nobody1305
            DB_PASSWORD=faizal1305
            DB_NAME=dumbmerch
            DB_PORT=5432 
            PORT=5000
    - name: "making dockerfile"
      copy:
        dest: "/home/nobody1305/be-dumbmerch/Dockerfile"
        content: |
          FROM golang:1.20-alpine
          WORKDIR /app
          COPY . /app
          RUN go install github.com/beego/bee/v2@latest
          #RUN go get -x ./
          #RUN go build
          RUN go mod download
          RUN go mod tidy 
          EXPOSE 5000
          CMD ["bee", "run"]

```
making docker compose
```
- become: true
  gather_facts: false
  hosts: appserver
  tasks:
    - name: "compose down"
      docker_compose:
        project_src: "/home/nobody1305"
        files:
          - docker-compose.yml
        state: absent
    - name: "making docker compose"
      copy:
        dest: "/home/nobody1305/docker-compose.yml"
        content: |
          version: "3.8"
          services:
            db:
               image: postgres
               restart: always
               container_name: db-dumbmerch
               expose:
                 - 5432
               ports:
                 - 5432:5432
               volumes:
                 - ~/postgresql:/var/lib/postgresql/data
               environment:
                 - POSTGRES_ROOT_PASSWORD=faizal1305
                 - POSTGRES_USER=nobody1305
                 - POSTGRES_PASSWORD=faizal1305
                 - POSTGRES_DATABASE=dumbmerch
            backend:
              depends_on:
                 - db
              build: ./be-dumbmerch
              image: nobody1305/be-dumbmerch
              container_name: be-dumbmerch
              stdin_open: true
              restart: unless-stopped
              ports:
                 - 5000:5000
            frontend:
              build: ./fe-dumbmerch
              image: nobody1305/fe-dumbmerch
              container_name: fe-dumbmerch
              stdin_open: true
              restart: unless-stopped
              ports:
                - 3000:3000
#    - name: "Install docker-compose python package"
#      ansible.builtin.pip:
#        name: docker-compose
    - name: "Create and start services"
      docker_compose:
        project_src: "/home/nobody1305"
        files:
          - docker-compose.yml
        recreate: always

```
![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/05e252c3-9359-4994-bc6d-7432961699c5)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/d5bc2c3e-f57b-4cd9-b070-b52443fac9d1)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/159d5243-55c7-478a-9cc1-1648eeb1c43a)

kita bisa melihat databasenya 

```
\c (masuk database)
\d (melihat table)
SELECT * FROM users; 
```

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/da34271d-3c81-4995-9d91-d827e9e56e11)

# CICD

buat kredensial di jenkins dan github private repo

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/52c76588-46fe-4fa4-b23d-dc7ff3a39ca3)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/be6c9fec-be59-4ea8-a907-ec4cf553eb52)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/56a74560-4f23-4039-8121-1dd3a7f6fe3f)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/cca60d9a-6d72-416a-aaaa-7f03b3fa9ee4)

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/d2b9997b-b3e0-4efe-a3a1-dad55b6d3bfb)

lakukan hal sama dengan backend

![image](https://github.com/fifa0903/devops17-finaltask-faizal/assets/132969781/5e571a83-de94-410e-b214-94d9c9148598)


