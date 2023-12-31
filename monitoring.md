
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

