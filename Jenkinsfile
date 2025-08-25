pipeline {
    agent any

    environment {
        WAR_FILE = "target/addressbook.war"
        TOMCAT_HOME = "/opt/tomcat9"
        INVENTORY = "addressbook_repo/inventory.ini"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "=== Checking out code from GitHub ==="
                git branch: 'main', url: 'https://github.com/Sathya252/milestone_practice.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo "=== Compiling and packaging with Maven ==="
                sh 'mvn clean package'
            }
        }

        stage('Install Tomcat 9 via Ansible') {
            steps {
                echo "=== Installing Tomcat 9 on Application Node via Ansible ==="
                writeFile file: 'tomcat.yml', text: '''
                - hosts: webservers
                  become: true
                  tasks:
                    - name: Install Java
                      apt:
                        name: openjdk-11-jre
                        state: present
                        update_cache: yes

                    - name: Install unzip
                      apt:
                        name: unzip
                        state: present

                    - name: Download Tomcat 9
                      get_url:
                        url: https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.108/bin/apache-tomcat-9.0.108.zip
                        dest: /tmp/apache-tomcat-9.0.108.zip

                    - name: Extract Tomcat
                      unarchive:
                        src: /tmp/apache-tomcat-9.0.108.zip
                        dest: /opt/
                        remote_src: yes

                    - name: Rename Tomcat folder
                      command: mv /opt/apache-tomcat-9.0.108 /opt/tomcat9
                      args:
                        creates: /opt/tomcat9

                    - name: Make Tomcat scripts executable
                      command: chmod +x /opt/tomcat9/bin/*.sh
                '''
                sh 'ansible-playbook -i ${INVENTORY} tomcat.yml'
            }
        }

        stage('Deploy WAR to Tomcat') {
            steps {
                echo "=== Deploying WAR file to Tomcat ==="
                sh 'ansible webservers -i ${INVENTORY} -m copy -a "src=${WAR_FILE} dest=${TOMCAT_HOME}/webapps/addressbook.war" --become'
                sh 'ansible webservers -i ${INVENTORY} -m shell -a "${TOMCAT_HOME}/bin/shutdown.sh || true" --become'
                sh 'ansible webservers -i ${INVENTORY} -m shell -a "${TOMCAT_HOME}/bin/startup.sh" --become'
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline finished successfully! AddressBook deployed on Tomcat."
        }
        failure {
            echo "❌ Pipeline failed. Check the logs for errors."
        }
    }
}
