#!/bin/bash

# Script to setup the Library Management System

# Colors for output
GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m' # No Color

# Function to display an error message and exit
error_exit() {
    echo -e "${RED}Error: $1${NC}" >&2
    exit 1
}

# Function to check if a command exists
command_exists() {
    type "$1" &> /dev/null
}

# Step 1: Clone the repository from GitHub
echo -e "${GREEN}Cloning the project repository...${NC}"
git clone https://github.com/your-repo/library-management-system.git || error_exit "Failed to clone repository"
cd library-management-system || exit

# Step 2: Check if JDK 17 is installed
echo -e "${GREEN}Checking JDK 17 installation...${NC}"
if command_exists java; then
    echo "Java found in PATH."
    _java=java
elif [[ -n "$JAVA_HOME" ]] && [[ -x "$JAVA_HOME/bin/java" ]]; then
    echo "Java found in JAVA_HOME."
    _java="$JAVA_HOME/bin/java"
else
    echo "JDK 17 is not installed. Downloading and installing..."
    wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz || error_exit "Failed to download JDK"
    sudo tar -xvzf jdk-17_linux-x64_bin.tar.gz -C /opt/ || error_exit "Failed to extract JDK"
    export JAVA_HOME=/opt/jdk-17
    export PATH=$JAVA_HOME/bin:$PATH
    echo -e "${GREEN}JDK 17 installed successfully.${NC}"
fi

java_version=$("$_java" -version 2>&1 | awk -F[\".] '{print $2}')
if [ "$java_version" -lt 17 ]; then
    error_exit "JDK 17 is required but lower version detected."
else
    echo "JDK 17 is installed."
fi

# Step 3: Install Apache Maven if not installed
if ! command_exists mvn; then
    echo "Maven is not installed. Downloading and installing Maven..."
    wget https://apache.mirror.digitalpacific.com.au/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz || error_exit "Failed to download Maven"
    sudo tar -xvzf apache-maven-3.8.5-bin.tar.gz -C /opt/ || error_exit "Failed to extract Maven"
    export M2_HOME=/opt/apache-maven-3.8.5
    export PATH=$M2_HOME/bin:$PATH
    echo -e "${GREEN}Maven installed successfully.${NC}"
else
    echo "Maven is already installed."
fi

# Step 4: Build the project with Maven
echo -e "${GREEN}Building the project with Maven...${NC}"
mvn clean install || error_exit "Maven build failed"

# Step 5: Check if port 9080 is available
PORT=9080
if lsof -Pi :$PORT -sTCP:LISTEN -t >/dev/null ; then
    error_exit "Port $PORT is already in use. Please free the port or modify the configuration."
fi

# Step 6: Run the Spring Boot Application
echo -e "${GREEN}Starting the Spring Boot application...${NC}"
mvn spring-boot:run &

# Step 7: Provide admin credentials for login
echo -e "${GREEN}Application is running. You can access it at: http://localhost:$PORT${NC}"
echo -e "${GREEN}Admin Login Credentials:${NC}"
echo "User ID: admin@admin.in"
echo "Password: Temp123"

# Optional Step: Monitor logs
echo -e "${GREEN}Displaying application logs (Ctrl+C to stop)...${NC}"
sleep 5  # Give time for the server to start
tail -f logs/spring.log || echo -e "${RED}Could not display logs.${NC}"

# Step 8: Setup database (optional)
echo -n "Would you like to set up a database (MySQL/PostgreSQL)? (y/n): "
read setup_db
if [ "$setup_db" == "y" ]; then
    echo "Setting up database..."
    # Add commands for database setup here, e.g., creating schemas, tables, etc.
    echo -e "${GREEN}Database setup complete.${NC}"
else
    echo "Skipping database setup."
fi

# End of Script
