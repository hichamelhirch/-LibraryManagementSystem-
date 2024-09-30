#!/bin/bash

# Script to setup the Library Management System

# Step 1: Clone the repository from GitHub
echo "Cloning the project repository..."
git clone https://github.com/your-repo/library-management-system.git
cd library-management-system || exit

# Step 2: Check if JDK 17 is installed
echo "Checking JDK 17 installation..."
if type -p java; then
    echo "Java found in PATH."
    _java=java
elif [[ -n "$JAVA_HOME" ]] && [[ -x "$JAVA_HOME/bin/java" ]]; then
    echo "Java found in JAVA_HOME."
    _java="$JAVA_HOME/bin/java"
else
    echo "JDK 17 is not installed. Downloading and installing..."
    wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz
    sudo tar -xvzf jdk-17_linux-x64_bin.tar.gz -C /opt/
    export JAVA_HOME=/opt/jdk-17
    export PATH=$JAVA_HOME/bin:$PATH
    echo "JDK 17 installed successfully."
fi

java_version=$("$_java" -version 2>&1 | awk -F[\".] '{print $2}')
if [ "$java_version" -lt 17 ]; then
    echo "JDK 17 is required but lower version detected."
    exit 1
else
    echo "JDK 17 is installed."
fi

# Step 3: Install Apache Maven if not installed
if ! type mvn > /dev/null; then
    echo "Maven is not installed. Downloading and installing Maven..."
    wget https://apache.mirror.digitalpacific.com.au/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz
    sudo tar -xvzf apache-maven-3.8.5-bin.tar.gz -C /opt/
    export M2_HOME=/opt/apache-maven-3.8.5
    export PATH=$M2_HOME/bin:$PATH
    echo "Maven installed successfully."
else
    echo "Maven is already installed."
fi

# Step 4: Build the project with Maven
echo "Building the project with Maven..."
mvn clean install

# Step 5: Run the Spring Boot Application
echo "Starting the Spring Boot application..."
mvn spring-boot:run &

# Step 6: Provide admin credentials for login
echo "Application is running. You can access it at: http://localhost:9080"
echo "Admin Login Credentials:"
echo "User ID: admin@admin.in"
echo "Password: Temp123"

# End of Script
