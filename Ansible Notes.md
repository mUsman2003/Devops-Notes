# Ansible Case Studies and Playbook Solutions

## Case Study 1: Web Server Deployment and Configuration

**Scenario**: A company needs to deploy and configure Apache web servers across multiple environments (dev, staging, production) with consistent configuration and SSL certificates.

### Playbook: `web-server-deployment.yml`

```yaml
---
- name: Deploy and Configure Apache Web Servers
  hosts: web_servers
  become: yes
  vars:
    apache_service: "{{ 'httpd' if ansible_os_family == 'RedHat' else 'apache2' }}"
    apache_conf_dir: "{{ '/etc/httpd/conf' if ansible_os_family == 'RedHat' else '/etc/apache2' }}"
    document_root: /var/www/html
    
  tasks:
    - name: Install Apache web server
      package:
        name: "{{ apache_service }}"
        state: present
        
    - name: Start and enable Apache service
      service:
        name: "{{ apache_service }}"
        state: started
        enabled: yes
        
    - name: Create document root directory
      file:
        path: "{{ document_root }}"
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
        
    - name: Deploy index.html
      template:
        src: index.html.j2
        dest: "{{ document_root }}/index.html"
        owner: www-data
        group: www-data
        mode: '0644'
      notify: restart apache
      
    - name: Configure virtual host
      template:
        src: vhost.conf.j2
        dest: "{{ apache_conf_dir }}/sites-available/{{ inventory_hostname }}.conf"
      notify: restart apache
      
    - name: Enable virtual host
      command: a2ensite {{ inventory_hostname }}.conf
      when: ansible_os_family == "Debian"
      notify: restart apache
      
    - name: Configure firewall for HTTP/HTTPS
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - '80'
        - '443'
      when: ansible_os_family == "Debian"
      
  handlers:
    - name: restart apache
      service:
        name: "{{ apache_service }}"
        state: restarted
```

**What this playbook does**:
- Installs Apache web server on multiple OS families (RedHat/Debian)
- Starts and enables the Apache service for automatic startup
- Creates proper directory structure with correct permissions
- Deploys customized index.html from a template
- Configures virtual hosts for each server
- Sets up firewall rules for web traffic
- Uses handlers to restart Apache only when configuration changes

---

## Case Study 2: Database Server Setup with Security

**Scenario**: Setting up MySQL database servers with security hardening, user management, and backup configuration.

### Playbook: `mysql-secure-setup.yml`

```yaml
---
- name: MySQL Database Server Setup and Security
  hosts: database_servers
  become: yes
  vars:
    mysql_root_password: "{{ vault_mysql_root_password }}"
    mysql_databases:
      - name: production_db
        encoding: utf8mb4
        collation: utf8mb4_unicode_ci
      - name: staging_db
        encoding: utf8mb4
        collation: utf8mb4_unicode_ci
    mysql_users:
      - name: app_user
        password: "{{ vault_app_user_password }}"
        privileges: "production_db.*:ALL"
        host: "%"
      - name: backup_user
        password: "{{ vault_backup_user_password }}"
        privileges: "*.*:SELECT,LOCK TABLES,SHOW VIEW,EVENT,TRIGGER"
        host: "localhost"

  tasks:
    - name: Install MySQL server
      package:
        name: 
          - mysql-server
          - python3-pymysql
        state: present
        
    - name: Start and enable MySQL service
      service:
        name: mysql
        state: started
        enabled: yes
        
    - name: Set MySQL root password
      mysql_user:
        name: root
        password: "{{ mysql_root_password }}"
        login_unix_socket: /var/run/mysqld/mysqld.sock
        
    - name: Remove anonymous MySQL users
      mysql_user:
        name: ''
        host_all: yes
        state: absent
        login_user: root
        login_password: "{{ mysql_root_password }}"
        
    - name: Remove MySQL test database
      mysql_db:
        name: test
        state: absent
        login_user: root
        login_password: "{{ mysql_root_password }}"
        
    - name: Create application databases
      mysql_db:
        name: "{{ item.name }}"
        encoding: "{{ item.encoding }}"
        collation: "{{ item.collation }}"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
      loop: "{{ mysql_databases }}"
      
    - name: Create MySQL users
      mysql_user:
        name: "{{ item.name }}"
        password: "{{ item.password }}"
        priv: "{{ item.privileges }}"
        host: "{{ item.host }}"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
      loop: "{{ mysql_users }}"
      
    - name: Configure MySQL for security
      lineinfile:
        path: /etc/mysql/mysql.conf.d/mysqld.cnf
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
      loop:
        - { regexp: '^bind-address', line: 'bind-address = 0.0.0.0' }
        - { regexp: '^max_connections', line: 'max_connections = 100' }
      notify: restart mysql
      
    - name: Setup MySQL backup script
      template:
        src: mysql-backup.sh.j2
        dest: /usr/local/bin/mysql-backup.sh
        mode: '0750'
        owner: root
        
    - name: Schedule daily backup
      cron:
        name: "MySQL daily backup"
        minute: "0"
        hour: "2"
        job: "/usr/local/bin/mysql-backup.sh"
        
  handlers:
    - name: restart mysql
      service:
        name: mysql
        state: restarted
```

**What this playbook does**:
- Installs MySQL server and required Python modules
- Secures MySQL installation by removing anonymous users and test database
- Creates application databases with proper encoding
- Sets up database users with specific privileges
- Configures MySQL security settings
- Implements automated backup solution with cron scheduling
- Uses Ansible Vault for password security

---

## Case Study 3: Docker Container Orchestration

**Scenario**: Deploying a multi-container application stack (web app + database + redis cache) across multiple Docker hosts.

### Playbook: `docker-app-stack.yml`

```yaml
---
- name: Deploy Multi-Container Application Stack
  hosts: docker_hosts
  become: yes
  vars:
    app_name: myapp
    app_version: latest
    networks:
      - name: "{{ app_name }}_network"
        driver: bridge
    volumes:
      - name: "{{ app_name }}_db_data"
      - name: "{{ app_name }}_redis_data"
    containers:
      - name: "{{ app_name }}_db"
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: "{{ vault_mysql_root_password }}"
          MYSQL_DATABASE: "{{ app_name }}"
          MYSQL_USER: appuser
          MYSQL_PASSWORD: "{{ vault_mysql_user_password }}"
        volumes:
          - "{{ app_name }}_db_data:/var/lib/mysql"
        networks:
          - "{{ app_name }}_network"
      - name: "{{ app_name }}_redis"
        image: redis:alpine
        volumes:
          - "{{ app_name }}_redis_data:/data"
        networks:
          - "{{ app_name }}_network"
      - name: "{{ app_name }}_web"
        image: "mycompany/{{ app_name }}:{{ app_version }}"
        ports:
          - "80:8000"
        env:
          DB_HOST: "{{ app_name }}_db"
          REDIS_HOST: "{{ app_name }}_redis"
          APP_ENV: production
        networks:
          - "{{ app_name }}_network"
        depends_on:
          - "{{ app_name }}_db"
          - "{{ app_name }}_redis"

  tasks:
    - name: Install Docker
      package:
        name: docker.io
        state: present
        
    - name: Install Docker Compose
      pip:
        name: docker-compose
        
    - name: Start and enable Docker service
      service:
        name: docker
        state: started
        enabled: yes
        
    - name: Create Docker networks
      docker_network:
        name: "{{ item.name }}"
        driver: "{{ item.driver }}"
      loop: "{{ networks }}"
      
    - name: Create Docker volumes
      docker_volume:
        name: "{{ item.name }}"
      loop: "{{ volumes }}"
      
    - name: Deploy database container
      docker_container:
        name: "{{ containers[0].name }}"
        image: "{{ containers[0].image }}"
        env: "{{ containers[0].env }}"
        volumes: "{{ containers[0].volumes }}"
        networks: "{{ containers[0].networks }}"
        restart_policy: unless-stopped
        state: started
        
    - name: Deploy Redis container
      docker_container:
        name: "{{ containers[1].name }}"
        image: "{{ containers[1].image }}"
        volumes: "{{ containers[1].volumes }}"
        networks: "{{ containers[1].networks }}"
        restart_policy: unless-stopped
        state: started
        
    - name: Wait for database to be ready
      wait_for:
        port: 3306
        host: "{{ containers[0].name }}"
        delay: 10
        
    - name: Deploy web application container
      docker_container:
        name: "{{ containers[2].name }}"
        image: "{{ containers[2].image }}"
        ports: "{{ containers[2].ports }}"
        env: "{{ containers[2].env }}"
        networks: "{{ containers[2].networks }}"
        restart_policy: unless-stopped
        state: started
        
    - name: Configure log rotation for containers
      template:
        src: docker-logrotate.j2
        dest: /etc/logrotate.d/docker-containers
```

**What this playbook does**:
- Installs Docker and Docker Compose on target hosts
- Creates isolated Docker networks for container communication
- Sets up persistent volumes for data storage
- Deploys containers in proper dependency order (database first, then app)
- Configures environment variables and networking
- Implements container restart policies for reliability
- Sets up log rotation to prevent disk space issues

---

## Case Study 4: System Monitoring Setup

**Scenario**: Setting up comprehensive monitoring with Prometheus, Grafana, and node exporters across infrastructure.

### Playbook: `monitoring-stack.yml`

```yaml
---
- name: Deploy Monitoring Stack
  hosts: monitoring_servers
  become: yes
  vars:
    prometheus_version: 2.40.0
    grafana_version: latest
    node_exporter_version: 1.5.0
    monitoring_user: prometheus
    
  tasks:
    - name: Create monitoring user
      user:
        name: "{{ monitoring_user }}"
        system: yes
        shell: /bin/false
        home: /var/lib/prometheus
        
    - name: Create Prometheus directories
      file:
        path: "{{ item }}"
        state: directory
        owner: "{{ monitoring_user }}"
        group: "{{ monitoring_user }}"
        mode: '0755'
      loop:
        - /etc/prometheus
        - /var/lib/prometheus
        - /opt/prometheus
        
    - name: Download and install Prometheus
      unarchive:
        src: "https://github.com/prometheus/prometheus/releases/download/v{{ prometheus_version }}/prometheus-{{ prometheus_version }}.linux-amd64.tar.gz"
        dest: /opt/prometheus
        remote_src: yes
        creates: "/opt/prometheus/prometheus-{{ prometheus_version }}.linux-amd64"
        owner: "{{ monitoring_user }}"
        group: "{{ monitoring_user }}"
        
    - name: Create Prometheus symlinks
      file:
        src: "/opt/prometheus/prometheus-{{ prometheus_version }}.linux-amd64/{{ item }}"
        dest: "/usr/local/bin/{{ item }}"
        state: link
      loop:
        - prometheus
        - promtool
        
    - name: Configure Prometheus
      template:
        src: prometheus.yml.j2
        dest: /etc/prometheus/prometheus.yml
        owner: "{{ monitoring_user }}"
        group: "{{ monitoring_user }}"
        mode: '0644'
      notify: restart prometheus
      
    - name: Create Prometheus systemd service
      template:
        src: prometheus.service.j2
        dest: /etc/systemd/system/prometheus.service
      notify:
        - reload systemd
        - restart prometheus
        
    - name: Install Grafana
      apt:
        deb: "https://dl.grafana.com/oss/release/grafana_{{ grafana_version }}_amd64.deb"
        state: present
      when: ansible_os_family == "Debian"
      
    - name: Configure Grafana
      template:
        src: grafana.ini.j2
        dest: /etc/grafana/grafana.ini
      notify: restart grafana
      
    - name: Start and enable services
      systemd:
        name: "{{ item }}"
        state: started
        enabled: yes
        daemon_reload: yes
      loop:
        - prometheus
        - grafana-server
        
  handlers:
    - name: reload systemd
      systemd:
        daemon_reload: yes
        
    - name: restart prometheus
      systemd:
        name: prometheus
        state: restarted
        
    - name: restart grafana
      systemd:
        name: grafana-server
        state: restarted

- name: Deploy Node Exporters
  hosts: all
  become: yes
  vars:
    node_exporter_version: 1.5.0
    
  tasks:
    - name: Download and install Node Exporter
      unarchive:
        src: "https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz"
        dest: /opt
        remote_src: yes
        creates: "/opt/node_exporter-{{ node_exporter_version }}.linux-amd64"
        
    - name: Create Node Exporter symlink
      file:
        src: "/opt/node_exporter-{{ node_exporter_version }}.linux-amd64/node_exporter"
        dest: /usr/local/bin/node_exporter
        state: link
        
    - name: Create Node Exporter service
      template:
        src: node_exporter.service.j2
        dest: /etc/systemd/system/node_exporter.service
      notify: restart node_exporter
      
    - name: Start and enable Node Exporter
      systemd:
        name: node_exporter
        state: started
        enabled: yes
        daemon_reload: yes
        
  handlers:
    - name: restart node_exporter
      systemd:
        name: node_exporter
        state: restarted
```

**What this playbook does**:
- Creates dedicated system users for security
- Downloads and installs Prometheus monitoring server
- Sets up Grafana for visualization dashboards
- Deploys Node Exporters on all servers for system metrics
- Configures systemd services for automatic startup
- Creates proper directory structure with correct permissions
- Uses templates for flexible configuration management

---

## Case Study 5: Automated Backup and Recovery

**Scenario**: Implementing automated backup solution for databases, application data, and system configurations across multiple servers.

### Playbook: `backup-automation.yml`

```yaml
---
- name: Automated Backup and Recovery System
  hosts: all
  become: yes
  vars:
    backup_user: backup
    backup_base_dir: /backup
    retention_days: 30
    backup_time: "02:00"
    
  tasks:
    - name: Create backup user
      user:
        name: "{{ backup_user }}"
        home: "{{ backup_base_dir }}"
        shell: /bin/bash
        system: yes
        
    - name: Create backup directories
      file:
        path: "{{ item }}"
        state: directory
        owner: "{{ backup_user }}"
        group: "{{ backup_user }}"
        mode: '0755'
      loop:
        - "{{ backup_base_dir }}"
        - "{{ backup_base_dir }}/databases"
        - "{{ backup_base_dir }}/files"
        - "{{ backup_base_dir }}/configs"
        - "{{ backup_base_dir }}/logs"
        
    - name: Install backup tools
      package:
        name:
          - rsync
          - gzip
          - tar
          - mysqldump
        state: present
        
    - name: Create database backup script
      template:
        src: db-backup.sh.j2
        dest: "{{ backup_base_dir }}/scripts/db-backup.sh"
        owner: "{{ backup_user }}"
        mode: '0750'
      when: "'database_servers' in group_names"
      
    - name: Create file backup script
      template:
        src: file-backup.sh.j2
        dest: "{{ backup_base_dir }}/scripts/file-backup.sh"
        owner: "{{ backup_user }}"
        mode: '0750'
        
    - name: Create config backup script
      template:
        src: config-backup.sh.j2
        dest: "{{ backup_base_dir }}/scripts/config-backup.sh"
        owner: "{{ backup_user }}"
        mode: '0750'
        
    - name: Create cleanup script
      template:
        src: backup-cleanup.sh.j2
        dest: "{{ backup_base_dir }}/scripts/cleanup.sh"
        owner: "{{ backup_user }}"
        mode: '0750'
        
    - name: Schedule database backups
      cron:
        name: "Database backup"
        minute: "0"
        hour: "{{ backup_time.split(':')[0] }}"
        job: "{{ backup_base_dir }}/scripts/db-backup.sh"
        user: "{{ backup_user }}"
      when: "'database_servers' in group_names"
      
    - name: Schedule file backups
      cron:
        name: "File backup"
        minute: "30"
        hour: "{{ backup_time.split(':')[0] }}"
        job: "{{ backup_base_dir }}/scripts/file-backup.sh"
        user: "{{ backup_user }}"
        
    - name: Schedule config backups
      cron:
        name: "Config backup"
        minute: "15"
        hour: "{{ backup_time.split(':')[0] }}"
        job: "{{ backup_base_dir }}/scripts/config-backup.sh"
        user: "{{ backup_user }}"
        
    - name: Schedule cleanup
      cron:
        name: "Backup cleanup"
        minute: "0"
        hour: "4"
        job: "{{ backup_base_dir }}/scripts/cleanup.sh"
        user: "{{ backup_user }}"
        
    - name: Create backup verification script
      template:
        src: verify-backups.sh.j2
        dest: "{{ backup_base_dir }}/scripts/verify-backups.sh"
        owner: "{{ backup_user }}"
        mode: '0750'
        
    - name: Setup backup monitoring
      template:
        src: backup-monitor.sh.j2
        dest: "{{ backup_base_dir }}/scripts/monitor.sh"
        owner: "{{ backup_user }}"
        mode: '0750'
```

**What this playbook does**:
- Creates dedicated backup user and directory structure
- Installs necessary backup tools (rsync, tar, mysqldump)
- Generates backup scripts from templates for different data types
- Schedules automated backups using cron at different times
- Implements retention policies to manage disk space
- Sets up backup verification and monitoring scripts
- Ensures proper permissions and security for backup operations

---

## Additional Use Cases Summary

### 1. **CI/CD Pipeline Integration**
- Deploying applications from Git repositories
- Managing environment-specific configurations
- Rolling updates and rollback procedures

### 2. **Security Hardening**
- System patching and updates
- User access management
- Firewall configuration
- SSL certificate deployment

### 3. **Load Balancer Configuration**
- HAProxy/Nginx setup
- Health checks and failover
- Dynamic backend server management

### 4. **Cloud Infrastructure Management**
- AWS/Azure resource provisioning
- Auto-scaling group management
- Storage and network configuration

### 5. **Compliance and Auditing**
- Security baseline enforcement
- Log aggregation setup
- Compliance reporting automation

Each playbook demonstrates key Ansible concepts like variables, templates, handlers, conditionals, loops, and error handling while solving real-world infrastructure challenges.
