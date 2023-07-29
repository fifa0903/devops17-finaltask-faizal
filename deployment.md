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
