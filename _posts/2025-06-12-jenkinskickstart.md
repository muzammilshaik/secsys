---
author: muju
title: "Jenkins kickstart"
description: ""
categories: ["Devops"]
tags: ["Linux", "Jenkins"]
image:
  path: /assets/vm/ct/jenkins/j.webp
  lqip: data:image/webp;base64,UklGRu4BAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSJ0AAAABgJpt27Lsxp1GYghI1iC6LMEALtndyURrHEzABCT53d3df/x93xEiYgKwVuntn31/7rU02MyLv8zW//g2cOszUsc6y4y4u8Z0SnZrBGD5mlG+m6F+mFHfqKozhqljFpM/Fu+PLC76LNqmf7o/HYp0W1JwS1SzMIAcVQ+A64ngtxkb9LQAIPY0h2t+xyCWP678HYuIEL36+XsbcbEKAFZQOCAqAQAAkAYAnQEqFAAUAD6RQJhJpaOiISgKqLASCWIAUwZN23wLJcIhjdjBr6VyUcPvXfMV3IUqkDGuSED0Gn15QAD+wgdremj5uMHLQ8/JZ3QDRler5lCxn2HSLL2YFDKDXPr08Lo+rLQtkRIazXnr+6ZS6zM6T/79ZTLj/NI3PZ7MkB8Oa5ysX6+WNjVqnXGtKU0q6fOvdPT0VTyDlzbv/zB6lHRg2ZHYg8d/iZwbeaxSzITYC3fxU8rnm2mJ8monz9Z+Wy9LEQCPBrTCfCiTSzJjbzPy3/fZuEvQaMADD+FoKY1TBSuX98WrKKWJ5kh4nQz45I8p0Rb9AmPDiJ8aHo6F2foZBVda3tzm/2fOUzj481foUt7x2xavkoVmVSz9gs0l3fJgKuwlkAAAAA==
published: true
hidden: false
toc: true
---

# 🧰 What is Jenkins?

Jenkins is an open-source automation server written in Java. It helps automate the parts of software development related to building, testing, and deploying, facilitating continuous integration (CI) and continuous delivery (CD).

Originally developed as Hudson, Jenkins has grown to support a vast plugin ecosystem and works well with all major tools in the DevOps ecosystem including Git, Docker, Kubernetes, Ansible, and many more.

# 🌐 Jenkins in the DevOps Lifecycle
In the DevOps pipeline, Jenkins plays a central role by automating:
- Code Integration (CI)
- Testing (unit/integration)
- Building artifacts
- Deployment (CD)
- Monitoring and post-deployment scripts

This automation reduces manual effort, increases speed, and improves consistency across development and production environments.

Example:
A developer pushes code to GitHub → Jenkins automatically triggers a build → runs tests → deploys to staging → notifies via Slack.

## ⏰ Jenkins Triggers Explained

Triggers are ways to automate builds based on events. Here's a detailed guide on how to set up different types of triggers in Jenkins:

### ✅ 1. Trigger Builds Remotely (e.g., from scripts)

This option allows you to trigger builds using a simple HTTP request:

1. In your job configuration, select "Trigger builds remotely"
2. Provide an authentication token (e.g., `token1`) - this should be unique
3. Jenkins will provide a URL in this format: `JENKINS_URL/job/jobname/build?token=TOKEN_NAME`
4. Replace with your actual values:
   - Example: `http://<server_ip>:8080/job/job4/build?token=token1`
5. You can trigger the build by:
   - Opening this URL in any browser
   - Using curl with API token: `curl -l -u admin:<apiToken> http://<publicIP>:8080/job/CloneRepo/build?token=token1`

### ✅ 2. GitHub Hook Trigger for GITScm Polling

Automatically trigger builds when changes are pushed to GitHub:

1. In Jenkins job configuration, select "GitHub hook trigger for GITScm polling"
2. Save the job configuration
3. Go to your GitHub repository settings
4. Select "Webhooks" on the left side
5. Delete any existing webhooks if necessary
6. Click "Create webhook" on the right side
7. Configure the webhook:
   - Payload URL: `http://your-jenkins-url/github-webhook/` (e.g., `http://<server_ip>:8080/github-webhook/`)
   - Content type: Select `application/json`
   - Secret: No value needed
   - For "Which events would you like to trigger this webhook?", select "Just the push event"
   - Ensure "Active" is selected
8. Click "Add webhook"

Now, whenever you make changes to your repository, Jenkins will automatically trigger a new build.

### ✅ 3. Build Periodically (Cron Jobs)

Schedule builds to run at specific times:

1. In job configuration, select "Build periodically"
2. Enter a cron expression (e.g., `*/2 * * * *` to run every 2 minutes)
3. Save the configuration

Jenkins will automatically generate builds according to the schedule you specified.

### ✅ 4. Poll SCM

This option checks for changes in your source code repository at regular intervals:

1. Configure your job with source code management (e.g., Git)
   - Example repository: `https://github.com/githubusername/repo`
2. Select "Poll SCM" under Build Triggers
3. Enter a cron expression (e.g., `* * * * *` to check every minute)

Unlike "Build periodically", this will only generate a build when changes are detected in the repository.

### ✅ 5. Upstream/Downstream Triggers
Chain jobs together:
- Upstream triggers downstream once completed
- Useful in multi-stage pipelines
```groovy
build job: 'downstream-job-name'
```

## 🧪 Parameters in Jenkins Jobs

Parameters let you customize job behavior during runtime.

### Freestyle Parameters

- Go to Configure → This project is parameterized
- Add parameters like:
  - String Parameter (e.g., ENVIRONMENT = staging)
  - Choice Parameter (e.g., BRANCH = dev/test/prod)

```bash
#!/bin/bash
echo "Deploying to $ENVIRONMENT"
```

You can also reference parameters in Pipeline scripts:

```
pipeline {
    agent any

    parameters {
        string(name: 'USERNAME', defaultValue: 'admin')
    }
    stages {
        stage('Echo Username') {
            steps {
                echo "Hello ${params.USERNAME}"
            }
        }
    }
}
```

### parameters in Pipeline

```groovy
pipeline {
    agent any

    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')
        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')

        // Additional parameters
        run(name: 'RUN_PARAM', description: 'Select a build to use')  // Select a build from another job
        file(name: 'FILE_PARAM', description: 'Upload a file')       // Upload a file
        credentials(name: 'CREDENTIALS_PARAM', description: 'Select credentials') // Select Jenkins stored credentials
        booleanParam(name: 'FLAG', defaultValue: false, description: 'A simple flag')
        string(name: 'EMAIL', defaultValue: '', description: 'User email address')
        text(name: 'NOTES', defaultValue: '', description: 'Additional notes')
    }

    stages {
        stage('Example') {
            steps {
                echo "Hello ${params.PERSON}"
                echo "Biography: ${params.BIOGRAPHY}"
                echo "Toggle: ${params.TOGGLE}"
                echo "Choice: ${params.CHOICE}"
                echo "Password: ${params.PASSWORD}"
                echo "Run param: ${params.RUN_PARAM}"
                echo "File param: ${params.FILE_PARAM}"
                echo "Credentials param: ${params.CREDENTIALS_PARAM}"
                echo "Flag: ${params.FLAG}"
                echo "Email: ${params.EMAIL}"
                echo "Notes: ${params.NOTES}"
            }
        }
    }
}
```

### 🗺️ Groovy Map Variables

Map variables are useful for structured data or lookups inside pipelines.

```groovy
// Script 1: Example Map Usage
pipeline {
    agent any

    stages {
        stage('Example') {
            steps {
                script {
                    def myMap = [
                        name: 'Alice',
                        city: 'New York',
                        job: 'Developer'
                    ]

                    echo "Name: ${myMap.name}"
                    echo "City: ${myMap['city']}"
                    echo "Job: ${myMap.job}"
                }
            }
        }
    }
}

```
```groovy
// Script 2: Deploy with Config Map
def config = [
  dev : "10.0.0.1",
  prod: "10.0.0.2"
]

pipeline {
    agent any
    parameters {
        choice(name: 'ENV', choices: ['dev', 'prod'], description: 'Select Environment')
    }
    stages {
        stage('Deploy') {
            steps {
                script {
                    def selected_ip = config[params.ENV]
                    echo "Deploying to IP: ${selected_ip}"
                }
            }
        }
    }
}
```


<!-- <div style="display: flex; gap: 20px;">

  <pre style="flex: 1; padding: 10px; overflow-x: auto;">
<code class="groovy">
// Script 1: Deploy with Config Map
def config = [
  dev : "10.0.0.1",
  prod: "10.0.0.2"
]

pipeline {
    agent any
    parameters {
        choice(name: 'ENV', choices: ['dev', 'prod'], description: 'Select Environment')
    }
    stages {
        stage('Deploy') {
            steps {
                script {
                    def selected_ip = config[params.ENV]
                    echo "Deploying to IP: ${selected_ip}"
                }
            }
        }
    }
}
</code>
  </pre>
</div>
<div>
  <pre style="flex: 1; padding: 10px; overflow-x: auto;">
<code class="groovy">
// Script 2: Example Map Usage
pipeline {
    agent any

    stages {
        stage('Example') {
            steps {
                script {
                    def myMap = [
                        name: 'Alice',
                        city: 'New York',
                        job: 'Developer'
                    ]

                    echo "Name: ${myMap.name}"
                    echo "City: ${myMap['city']}"
                    echo "Job: ${myMap.job}"
                }
            }
        }
    }
}
</code>
  </pre>

</div> -->

## Pipeline Project 

Jenkins is a core DevOps tool that automates the CI/CD lifecycle. Whether you're deploying a basic Freestyle job or a full-blown multibranch pipeline, Jenkins supports it all.

Mastering Jenkins triggers, parameters, and build types will give you a significant advantage in automating your delivery processes.

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello, Jenkins!'
            }
            post {
                always {
                    echo 'This will always run after the stage.'
                }
                success {
                    echo 'This runs only if the stage succeeds.'
                }
                failure {
                    echo 'This runs only if the stage fails.'
                }
            }
        }
    }
}
```

<!-- SweetAlert2 CDN -->
<script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>

<style>
  #copyBtn {
    position: relative;
    padding: 12px 30px;
    background-color: #2c2c2e; /* Dark gray background */
    color: #e0e0e0; /* Light gray text */
    font-weight: 600;
    font-size: 1rem;
    border: 1.5px solid #444; /* Subtle border */
    border-radius: 12px; /* Rounded corners but less pill-shaped */
    cursor: pointer;
    box-shadow: 0 2px 6px rgba(0,0,0,0.8);
    transition: background-color 0.3s ease, border-color 0.3s ease, transform 0.2s ease;
    user-select: none;
    display: inline-flex;
    align-items: center;
    gap: 8px;
  }

  #copyBtn:hover {
    background-color: #3a3a3c; /* Slightly lighter dark on hover */
    border-color: #666; /* Lighten border */
    transform: scale(1.04);
  }

  #copyBtn:active {
    background-color: #242426; /* Darker on click */
    border-color: #555;
    transform: scale(0.98);
  }

  #copyBtn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
</style>

<button id="copyBtn">
  📋 Advanced
</button>




<!-- Hidden Jenkins Pipeline Snippet -->
<pre id="pipelineCode" style="display:none;">
pipeline {
    // 🧭 Define the default execution environment
    agent any
    // agent none → You must define agents at the stage level
    // agent { label 'linux-node' } → Run on a specific node
    // agent { docker 'node:14' } → Run inside a Docker container

    // ⏰ Triggers to automatically start the pipeline
    triggers {
        pollSCM('H/5 * * * *')           // Poll SCM every 5 mins
        cron('H H(0-7) * * 1-5')         // Run on weekdays during early hours
    }

    // 🌍 Global environment variables
    environment {
        GLOBAL_VAR = 'global_value'
        API_KEY = credentials('my-api-key-id')  // From Jenkins credentials
    }

    // 🔧 Parameters to allow input when triggering builds manually
    parameters {
        string(name: 'USERNAME', defaultValue: 'admin', description: 'Enter your username')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
        choice(name: 'ENV', choices: ['dev', 'qa', 'prod'], description: 'Select environment')
        password(name: 'SECRET', defaultValue: '', description: 'Enter secret')
    }

    // ⚙️ Pipeline-level options
    options {
        timeout(time: 1, unit: 'HOURS')       // Abort if pipeline exceeds 1 hour
        retry(2)                              // Retry failed pipeline runs
        timestamps()                          // Show timestamps in logs
        skipDefaultCheckout()                 // Prevent automatic SCM checkout
        disableConcurrentBuilds()             // Only one build at a time
    }

    // 🧰 Tools needed in the pipeline (installed via Jenkins)
    tools {
        maven 'Maven 3.8.6'
        jdk 'JDK 11'
        nodejs 'NodeJS 16'
    }

    // 📦 All pipeline execution happens inside these stages
    stages {

        stage('Checkout') {
            steps {
                checkout scm
                // or: git 'https://github.com/user/repo.git'
            }
        }

        stage('Build') {
            agent { label 'build-node' }  // Optional stage-specific agent
            environment {
                BUILD_ENV = 'production'
            }
            when {
                branch 'main'  // Run only for main branch
            }
            steps {
                echo "Building as $params.USERNAME"
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'Build successful ✅'
                }
                failure {
                    echo 'Build failed ❌'
                }
                always {
                    echo 'Build stage completed.'
                }
            }
        }

        stage('Test') {
            when {
                expression { return params.RUN_TESTS == true }
            }
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            parallel {
                stage('Deploy to Dev') {
                    when {
                        branch 'dev'
                    }
                    steps {
                        echo 'Deploying to development server...'
                        // deploy commands here
                    }
                }

                stage('Deploy to QA') {
                    when {
                        branch 'qa'
                    }
                    steps {
                        echo 'Deploying to QA server...'
                        // deploy commands here
                    }
                }

                stage('Deploy to Prod') {
                    when {
                        branch 'main'
                    }
                    steps {
                        echo 'Deploying to Production...'
                        // deploy commands here
                    }
                }
            }
        }
    }

    // 📜 Post actions for the entire pipeline
    post {
        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
        success {
            echo '🎉 Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
        unstable {
            echo '⚠️ Pipeline marked as unstable.'
        }
        changed {
            echo '🔄 Pipeline result has changed compared to the previous run.'
        }
    }
}

</pre>

<script>
  const copyBtn = document.getElementById('copyBtn');
  const pipelineCode = document.getElementById('pipelineCode');

  copyBtn.addEventListener('click', async () => {
    if (!pipelineCode) {
      Swal.fire({
        icon: 'error',
        title: 'Oops!',
        text: 'No code found to copy.',
        toast: true,
        position: 'top-end',
        showConfirmButton: false,
        timer: 2000,
        background: '#4b2e83',  // Dark purple background consistent with dark theme
        color: 'white',
        iconColor: '#f87171'
      });
      return;
    }

    try {
      copyBtn.disabled = true;
      copyBtn.style.cursor = 'not-allowed';
      copyBtn.textContent = '⏳ Copying...';  // Keep simple text for dark theme

      await navigator.clipboard.writeText(pipelineCode.innerText);

      Swal.fire({
        icon: 'success',
        title: 'Copied!',
        text: 'Jenkins pipeline snippet copied to clipboard.',
        toast: true,
        position: 'top-end',
        showConfirmButton: false,
        timer: 2000,
        timerProgressBar: true,
        background: '#2c2c2e',  // Dark gray background matching button
        color: 'white',
        iconColor: '#d1d5db'
      });
    } catch (err) {
      copyBtn.classList.add('shake');
      Swal.fire({
        icon: 'error',
        title: 'Oops!',
        text: 'Failed to copy to clipboard.',
        toast: true,
        position: 'top-end',
        showConfirmButton: false,
        timer: 2000,
        background: '#4b2e83',
        color: 'white',
        iconColor: '#f87171'
      });
      setTimeout(() => copyBtn.classList.remove('shake'), 500);
    } finally {
      copyBtn.disabled = false;
      copyBtn.style.cursor = 'pointer';
      copyBtn.textContent = '📋 Advanced';  // Restore original button text
    }
  });
</script>


## 🔌 Jenkins Plugins

Jenkins' power comes from its extensive plugin ecosystem. Here's how to install and use plugins effectively.

There are two ways to install Jenkins plugins:

### Method 1: Through the Jenkins UI

1. Navigate to **Manage Jenkins** → **Manage Plugins**
2. Click on the **Available** tab
3. Use the search box to find your desired plugin
4. Check the box next to the plugin name
5. Click **Install without restart** or **Download now and install after restart**
6. If needed, restart Jenkins after installation

### Method 2: Using Jenkins CLI

```bash
java -jar jenkins-cli.jar -s http://your-jenkins-url/ install-plugin plugin-name
```

### Warnings Plugin Installation

The Warnings Next Generation plugin is a powerful tool for static code analysis that can parse and visualize warnings from various tools.

1. Go to **Manage Jenkins** → **Manage Plugins** → **Available**
2. Search for "Warnings Next Generation"
3. Check the box and click **Install without restart**

### Maven with Warnings Plugin

Here's an example of integrating the Warnings plugin in a pipeline:

```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Static Analysis') {
            steps {
                // Run static analysis tools
                sh 'mvn checkstyle:checkstyle pmd:pmd spotbugs:spotbugs'
            }
            post {
                always {
                    // Process the results using Warnings NG plugin
                    recordIssues(
                        tools: [
                            checkStyle(pattern: '**/checkstyle-result.xml'),
                            pmdParser(pattern: '**/pmd.xml'),
                            spotBugs(pattern: '**/spotbugsXml.xml')
                        ],
                        qualityGates: [[threshold: 1, type: 'TOTAL', unstable: true]],
                        healthy: 10,
                        unhealthy: 100,
                        minimumSeverity: 'HIGH'
                    )
                }
            }
        }
    }
}
```

### Python with Warnings Plugin

```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Python') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pip install pylint flake8'
            }
        }
        
        stage('Static Analysis') {
            steps {
                sh 'pylint --output-format=parseable --reports=no *.py > pylint.log || true'
                sh 'flake8 --format=pylint *.py > flake8.log || true'
            }
            post {
                always {
                    recordIssues(
                        tools: [
                            pyLint(pattern: 'pylint.log'),
                            flake8(pattern: 'flake8.log')
                        ],
                        qualityGates: [[threshold: 10, type: 'TOTAL', unstable: true]]
                    )
                }
            }
        }
        
        stage('Test') {
            steps {
                sh 'python -m pytest --junitxml=test-results.xml'
            }
            post {
                always {
                    junit 'test-results.xml'
                }
            }
        }
    }
}
```

## 🔄 Static vs. Dynamic Jenkins Agents

| Feature/Aspect              | Static Agents                                              | Dynamic Agents                                                     |
| --------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------ |
| **Availability**            | Always connected and available                             | Provisioned on-demand, terminated after use                        |
| **Startup Time**            | No startup delay                                           | Slight delay due to provisioning                                   |
| **Resource Usage**          | Less efficient – idle resources                            | More efficient – uses resources only when needed                   |
| **Environment Consistency** | Can drift over time                                        | Fresh and consistent for each build                                |
| **Scalability**             | Limited – manual effort needed                             | High – can scale automatically based on demand                     |
| **Maintenance**             | Manual setup and maintenance required                      | Requires infrastructure for provisioning (e.g., cloud, Kubernetes) |
| **Troubleshooting**         | Easier – persistent environment                            | Harder – ephemeral environments                                    |
| **Use Case Fit**            | Predictable workloads, complex setup, specialized hardware | Varying workloads, cloud-native environments, CI/CD pipelines      |

## 🐳 Jenkins Pipelines with Docker Agent

Docker agents allow you to run your pipeline steps inside Docker containers, providing isolated and consistent environments for your builds.

### Setting Up Docker Integration

#### Prerequisites

1. Connect to your Jenkins server via terminal:
   ```bash
   sudo su -
   apt-get update && apt-get install docker.io
   ```

2. Install the Docker Pipeline plugin:
   - Navigate to **Manage Jenkins** → **Manage Plugins** → **Available**
   - Search for "Docker Pipeline"
   - Select it and click **Install**
   - After installation, restart Jenkins:
     ```bash
     systemctl restart jenkins
     ```

3. Configure Docker permissions:
   ```bash
   # Add jenkins user to docker group
   usermod -aG docker jenkins
   # Restart Jenkins service
   systemctl restart jenkins
   ```

### Maven Docker Agent

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.8.6-openjdk-11'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B clean package'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
            }
        }
    }
}
```

### Python Docker Agent

```groovy
pipeline {
    agent {
        docker {
            image 'python:3.9'
            args '-v ${WORKSPACE}:/app -w /app'
        }
    }
    
    stages {
        stage('Setup') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pip install pytest pytest-cov'
            }
        }
        
        stage('Test') {
            steps {
                sh 'python -m pytest --cov=. --cov-report=xml'
            }
            post {
                always {
                    junit 'test-results.xml'
                    recordCoverage(
                        tools: [[parser: 'COBERTURA', pattern: 'coverage.xml']],
                        qualityGates: [[threshold: 80, metric: 'LINE', unstable: true]]
                    )
                }
            }
        }
    }
}
```
## Jenkins Pipelines with windows Agent

Follow these easy steps to add and configure a Jenkins slave (agent) on a Windows machine:

### Step 1: Configure Agent on Jenkins Master

1. Go to Manage Jenkins → Nodes → New Node
2. Enter a Node name, select Permanent Agent, click Create
3. Fill in:
    - Remote root directory → Path to an empty folder on agent machine (e.g., C:\Jenkins)
    - Label → Optional
    - Launch method → Launch agent by connecting it to the controller
    - Availability → Keep this agent online as much as possible
4. save

![jenkins](/assets/devops/jenkins/jenkins_Settings.png){: width="800" height="300" }

### Step 2: Connect Agent from Windows Machine

1. Open the newly created node page → Copy the two Windows command lines under “Run from agent command line”
2. Open CMD as Administrator and run the command
```bat
cd C:\Jenkins
```
Paste and execute both commands
3. This will download agent.jar and connect the agent (temporarily)

### Step 3: Run Agent as a Windows Service

1. Download and setup [WinSW](https://github.com/winsw/winsw){:target="_blank"}
    - Download WinSW-x64.exe → Rename to jenkins-agent.exe
    - Download sample jenkins.xml → Rename to jenkins-agent.xml
    - Place both in the Jenkins folder (path from step 4)
2. Edit jenkins-agent.xml:
    - Set <id> and <name> to jenkins-agent
    - Replace <arguments> content with the second command (excluding java)
3. In CMD (as Admin), run:
```bat
cd C:\Jenkins
jenkins-agent.exe install
Get-Service jenkins-agent
Start-Service jenkins-agent
```
![jenkins](/assets/devops/jenkins/jenkins1.png){: width="800" height="300" }

## Docker Multi-Stage Example

```groovy
pipeline {
    agent none
    
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:14'
                    args '-v ${WORKSPACE}:/app -w /app'
                }
            }
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            agent {
                docker {
                    image 'node:14'
                    args '-v ${WORKSPACE}:/app -w /app'
                }
            }
            steps {
                sh 'npm test'
            }
        }
        
        stage('Deploy') {
            agent {
                docker {
                    image 'amazon/aws-cli:latest'
                    args '--entrypoint=""'
                }
            }
            steps {
                sh 'aws s3 sync ./build s3://my-bucket/'
            }
        }
    }
}
```

## 📢 Jenkins Notifications

Notifications keep your team informed about build and deployment status. Here's how to set up different notification systems.

### Slack Integration

#### Installing Slack Plugin

1. Go to **Manage Jenkins** → **Manage Plugins** → **Available**
2. Search for "Slack Notification"
3. Install the plugin and restart Jenkins if needed

#### Configuring Slack Integration

1. Create a Slack app and get the token:
   - Go to https://api.slack.com/apps
   - Click **Create New App** → **From scratch**
   - Name your app (e.g., "Jenkins Notifications") and select your workspace
   - Navigate to **OAuth & Permissions** → **Bot Token Scopes**
   - Add scopes: `chat:write`, `files:write`, `channels:read`
   - Install the app to your workspace
   - Copy the **Bot User OAuth Token**

2. Configure Jenkins:
   - Go to **Manage Jenkins** → **Configure System**
   - Find the **Slack** section
   - Enter your Workspace name
   - Add credentials (Secret text) with the Bot User OAuth Token
   - Specify default channel (e.g., `#jenkins-builds`)
   - Test the connection

#### Using Slack in Pipeline

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
    }
    
    post {
        success {
            slackSend(
                color: 'good',
                message: "✅ Build Successful: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "❌ Build Failed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }
        unstable {
            slackSend(
                color: 'warning',
                message: "⚠️ Build Unstable: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
            )
        }
    }
}
```

### Discord Integration

#### Installing Discord Plugin

1. Go to **Manage Jenkins** → **Manage Plugins** → **Available**
2. Search for "Discord Notifier"
3. Install the plugin and restart Jenkins if needed

#### Configuring Discord Integration

1. Create a Discord webhook:
   - Open Discord and go to the server where you want to receive notifications
   - Go to **Server Settings** → **Integrations** → **Webhooks**
   - Click **New Webhook**
   - Name it (e.g., "Jenkins Notifications")
   - Select the channel for notifications
   - Copy the Webhook URL

2. Configure Jenkins:
   - Go to **Manage Jenkins** → **Configure System**
   - Find the **Discord Notifier** section
   - Enter the Webhook URL
   - Configure other settings as needed

#### Using Discord in Pipeline

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
    }
    
    post {
        always {
            discordSend(
                description: "Jenkins Build ${currentBuild.currentResult}",
                link: env.BUILD_URL,
                result: currentBuild.currentResult,
                title: "${env.JOB_NAME} #${env.BUILD_NUMBER}",
                webhookURL: 'https://discord.com/api/webhooks/your-webhook-url'
            )
        }
    }
}
```

## 🔍 SonarQube Integration with Jenkins

SonarQube provides continuous code quality inspection. Here's how to integrate it with Jenkins.

### Setting Up SonarQube

#### Installing SonarQube

1. Using Docker (recommended):
   ```bash
   docker run -d --name sonarqube -p 9000:9000 sonarqube:latest
   ```

2. Access SonarQube at `http://your-server:9000`
   - Default credentials: admin/admin
   - You'll be prompted to change the password on first login

#### Creating a SonarQube Project and Token

1. Log in to SonarQube
2. Go to **Administration** → **Projects** → **Create Project**
3. Enter a project key and name
4. Go to **Administration** → **Security** → **Users**
5. Click on your username → **Tokens** → **Generate**
6. Name your token (e.g., "jenkins-integration") and click **Generate**
7. Copy the generated token (you won't be able to see it again)

### Jenkins Configuration

#### Installing SonarQube Scanner Plugin

1. Go to **Manage Jenkins** → **Manage Plugins** → **Available**
2. Search for "SonarQube Scanner"
3. Install the plugin and restart Jenkins if needed

#### Configuring SonarQube in Jenkins

1. Go to **Manage Jenkins** → **Configure System**
2. Find the **SonarQube servers** section
3. Click **Add SonarQube**
4. Enter a name (e.g., "SonarQube")
5. Enter the SonarQube URL (e.g., `http://your-server:9000`)
6. Add the SonarQube authentication token as a Jenkins credential
7. Select the credential in the dropdown
8. Save the configuration

#### Adding SonarQube Scanner Tool

1. Go to **Manage Jenkins** → **Global Tool Configuration**
2. Find the **SonarQube Scanner** section
3. Click **Add SonarQube Scanner**
4. Enter a name (e.g., "SonarQube Scanner")
5. Choose **Install automatically** or specify the path to your installation
6. Save the configuration

### SonarQube in maven

```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8.6'
        jdk 'JDK 11'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=my-project -Dsonar.projectName="My Project"'
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
```

### SonarQube with Python

```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=python-project \
                        -Dsonar.projectName="Python Project" \
                        -Dsonar.sources=. \
                        -Dsonar.python.coverage.reportPaths=coverage.xml \
                        -Dsonar.python.xunit.reportPath=test-results.xml
                    '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
```

## 🔄 Backup of Jenkins Server
Regular backups are essential for Jenkins high availability and disaster recovery.

1. **JENKINS_HOME directory**: Contains all configuration, job definitions, build history, and plugins
2. **Global configuration files**: `config.xml`, `credentials.xml`, etc.
3. **Job configurations**: Located in `$JENKINS_HOME/jobs/`
4. **User content**: Located in `$JENKINS_HOME/users/`
5. **Plugins**: Located in `$JENKINS_HOME/plugins/`

### Method 1: File System Backup

1. Stop Jenkins service:
   ```bash
   systemctl stop jenkins
   ```

2. Backup the JENKINS_HOME directory:
   ```bash
   tar -czf jenkins_backup_$(date +%Y%m%d).tar.gz /var/lib/jenkins
   ```

3. Start Jenkins service:
   ```bash
   systemctl start jenkins
   ```

4. Transfer the backup to a secure location:
   ```bash
   scp jenkins_backup_*.tar.gz backup-server:/backup/jenkins/
   ```

### Method 2: Using Jenkins Backup Plugin

1. Install the "Backup Plugin":
   - Go to **Manage Jenkins** → **Manage Plugins** → **Available**
   - Search for "Backup Plugin"
   - Install the plugin and restart Jenkins if needed

2. Configure the backup:
   - Go to **Manage Jenkins** → **Backup Manager**
   - Configure backup settings:
     - Backup directory
     - Backup contents (jobs, plugins, configurations)
     - Backup schedule
     - Retention policy

3. Run a manual backup or wait for the scheduled backup

### Method 3: Using ThinBackup Plugin

1. Install the "ThinBackup Plugin":
   - Go to **Manage Jenkins** → **Manage Plugins** → **Available**
   - Search for "ThinBackup"
   - Install the plugin and restart Jenkins if needed

2. Configure ThinBackup:
   - Go to **Manage Jenkins** → **ThinBackup**
   - Set the backup directory
   - Configure backup settings:
     - Backup schedule
     - What to backup (configurations, plugins, build results)
     - Retention policy

3. Run a manual backup or wait for the scheduled backup

### Automated Backup Script

Create a script for automated backups:

```bash
#!/bin/bash

# Configuration
JENKINS_HOME="/var/lib/jenkins"
BACKUP_DIR="/backup/jenkins"
BACKUP_COUNT=7  # Number of backups to keep

# Create backup filename with date
BACKUP_FILE="$BACKUP_DIR/jenkins_backup_$(date +%Y%m%d).tar.gz"

# Ensure backup directory exists
mkdir -p $BACKUP_DIR

# Stop Jenkins service
systemctl stop jenkins

# Create backup
tar -czf $BACKUP_FILE $JENKINS_HOME

# Start Jenkins service
systemctl start jenkins

# Remove old backups, keeping only the most recent $BACKUP_COUNT
ls -t $BACKUP_DIR/jenkins_backup_*.tar.gz | tail -n +$((BACKUP_COUNT+1)) | xargs -r rm

# Log the backup
echo "Jenkins backup completed: $BACKUP_FILE" >> $BACKUP_DIR/backup.log
```

Add this script to crontab to run automatically:

```bash
# Edit crontab
crontab -e

# Add this line to run backup every Sunday at 2 AM
0 2 * * 0 /path/to/jenkins_backup.sh
```

### Restoring from Backup

1. Stop Jenkins service:
   ```bash
   systemctl stop jenkins
   ```

2. Remove or rename the current Jenkins home directory:
   ```bash
   mv /var/lib/jenkins /var/lib/jenkins.old
   ```

3. Extract the backup:
   ```bash
   mkdir -p /var/lib/jenkins
   tar -xzf jenkins_backup_YYYYMMDD.tar.gz -C /
   ```

4. Fix permissions:
   ```bash
   chown -R jenkins:jenkins /var/lib/jenkins
   ```

5. Start Jenkins service:
   ```bash
   systemctl start jenkins
   ```

## 📚 Shared Libraries in Jenkins

Shared Libraries allow you to define reusable code that can be used across multiple Jenkins pipelines.

### What are Shared Libraries?

Shared Libraries are collections of Groovy scripts that can be defined once and used in multiple Jenkins pipelines. They help in:

- Reducing code duplication
- Standardizing pipeline practices
- Centralizing common functionality
- Making pipelines more maintainable

### Setting Up a Shared Library

1. Create a Git repository with the following structure:
   ```
   ├── src                     # Groovy source files
   │   └── org/example/
   ├── vars                    # Global variables/functions
   │   ├── buildApp.groovy
   │   └── deployApp.groovy
   └── resources               # Non-Groovy files
   ```

2. Configure the library in Jenkins:
   - Go to **Manage Jenkins** → **Configure System**
   - Scroll to **Global Pipeline Libraries**
   - Click **Add**
   - Enter a name (e.g., "shared-library")
   - Set the default version (e.g., "main")
   - Specify the retrieval method (e.g., "Modern SCM" → Git)
   - Enter your repository URL
   - Save the configuration

### Using Shared Libraries in Pipelines

```groovy
// Import the library
@Library('shared-library')_

// Use functions from the library
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                // Call a function from the shared library
                buildApp()
            }
        }
        
        stage('Deploy') {
            steps {
                // Call another function with parameters
                deployApp(env: 'production', region: 'us-west-2')
            }
        }
    }
}
```

### Example Shared Library Function

In your shared library repository, create a file `vars/buildApp.groovy`:

```groovy
// vars/buildApp.groovy
def call(Map config = [:]) {
    // Default configuration
    def settings = [
        tool: 'maven',
        args: '-B clean package'
    ] << config
    
    // Execute the build based on the tool
    if (settings.tool == 'maven') {
        sh "mvn ${settings.args}"
    } else if (settings.tool == 'gradle') {
        sh "gradle ${settings.args}"
    } else {
        error "Unsupported build tool: ${settings.tool}"
    }
}
```

## 🔒 Securing Jenkins

Security is critical for Jenkins as it often has access to sensitive codebases and deployment environments.

### Basic Security Measures

1. **Use HTTPS/SSL**:
   - Secure your Jenkins instance with SSL certificates
   - Prevent man-in-the-middle attacks and credential theft

2. **Authentication and Authorization**:
   - Use Jenkins' built-in user database or integrate with LDAP/AD
   - Implement role-based access control (RBAC)
   - Consider using the Role-based Authorization Strategy plugin

3. **Secure Credentials Management**:
   - Use the Credentials plugin to store sensitive information
   - Consider using external secret managers like HashiCorp Vault
   - Never hardcode credentials in pipeline scripts

4. **Keep Jenkins Updated**:
   - Regularly update Jenkins core and plugins
   - Subscribe to the Jenkins security mailing list

### Setting Up SSL for Jenkins

1. Generate SSL keys:
   ```bash
   openssl genrsa -out jenkins.key 2048
   openssl req -new -key jenkins.key -out jenkins.csr
   openssl x509 -req -days 365 -in jenkins.csr -signkey jenkins.key -out jenkins.crt
   ```

2. Package the keys into a keystore:
   ```bash
   openssl pkcs12 -export -out jenkins.p12 -inkey jenkins.key -in jenkins.crt -name jenkins
   ```

3. Import into a JKS keystore:
   ```bash
   keytool -importkeystore -srckeystore jenkins.p12 -srcstoretype PKCS12 -destkeystore jenkins.jks
   ```

4. Copy the JKS file to the Jenkins directory:
   ```bash
   cp jenkins.jks /var/lib/jenkins/
   ```

5. Configure Jenkins to use HTTPS:
   - Edit `/etc/default/jenkins` or `/etc/sysconfig/jenkins`
   - Add/modify these lines:
     ```
     JENKINS_ARGS="--httpPort=-1 --httpsPort=8443 --httpsKeyStore=/var/lib/jenkins/jenkins.jks --httpsKeyStorePassword=your_password"
     ```
   - Restart Jenkins: `systemctl restart jenkins`

## 🐍 Advanced Python Pipeline Example

Here's a comprehensive Python pipeline that includes environment setup, testing, code quality checks, and reporting:

```groovy
pipeline {
    agent {
        docker {
            image 'python:3.11-slim'
            args '-v ${WORKSPACE}:/app -w /app'
        }
    }
    
    environment {
        PYTHONPATH = "${WORKSPACE}"
        VIRTUAL_ENV = "${WORKSPACE}/.venv"
        PATH = "${VIRTUAL_ENV}/bin:${PATH}"
    }
    
    stages {
        stage('Checkout code') {
            steps {
                checkout scm
            }
        }
        
        stage('Set Up Environment') {
            steps {
                sh '''
                    python -m venv .venv
                    . .venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pip install pytest pytest-cov flake8 pylint
                '''
            }
        }
        
        stage('Code Review') {
            steps {
                sh '''
                    . .venv/bin/activate
                    flake8 --max-line-length=100 --exclude=.venv,migrations .
                '''
            }
            post {
                always {
                    recordIssues(
                        tools: [flake8(pattern: 'flake8_report.txt')],
                        qualityGates: [[threshold: 10, type: 'TOTAL', unstable: true]]
                    )
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                sh '''
                    . .venv/bin/activate
                    pytest --junitxml=test-results.xml --cov=. --cov-report=xml --cov-report=html
                '''
            }
            post {
                always {
                    junit 'test-results.xml'
                    recordCoverage(
                        tools: [[parser: 'COBERTURA', pattern: 'coverage.xml']],
                        qualityGates: [[threshold: 80, metric: 'LINE', unstable: true]]
                    )
                }
            }
        }
        
        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'htmlcov/**', allowEmptyArchive: true
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
    }
}
```

## 🔄 Mini-Project: Integrated Jenkins Pipeline

Here's a comprehensive pipeline that integrates Jenkins with Maven, SonarQube, and Slack notifications:

```groovy
pipeline {
  agent any

  parameters {
    choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'], description: 'Select deployment environment')
  }

  environment {
    MVN_CMD = "mvn"
    SONARQUBE_SERVER = 'SonarQubeServer'
    SLACK_CHANNEL = '#ci'
  }

  options {
    timeout(time: 30, unit: 'MINUTES')
  }

  triggers {
    pollSCM('H/5 * * * *')
  }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/your-org/your-repo.git'
      }
    }

    stage('Build & Test') {
      steps {
        sh "${MVN_CMD} clean package"
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("${SONARQUBE_SERVER}") {
          sh "${MVN_CMD} sonar:sonar"
        }
      }
    }

    stage('Deploy') {
      when {
        expression {
          // Deploy only for dev, staging, or prod parameter
          return params.DEPLOY_ENV in ['dev', 'staging', 'prod']
        }
      }
      steps {
        script {
          if (params.DEPLOY_ENV == 'dev') {
            echo "Deploying to Development environment..."
            // Add your dev deployment commands here, e.g.:
            // sh 'ssh user@dev-server "deploy-dev-script.sh"'
          } else if (params.DEPLOY_ENV == 'staging') {
            echo "Deploying to Staging environment..."
            // Add your staging deployment commands here, e.g.:
            // sh 'ssh user@staging-server "deploy-staging-script.sh"'
          } else if (params.DEPLOY_ENV == 'prod') {
            echo "Deploying to Production environment..."
            // Add your production deployment commands here, e.g.:
            // sh 'ssh user@prod-server "deploy-prod-script.sh"'
          }
        }
      }
    }
  }

  post {
    success {
      slackSend(channel: "${SLACK_CHANNEL}", message: "Build & Deploy Success: ${env.JOB_NAME} #${env.BUILD_NUMBER} to ${params.DEPLOY_ENV}")
    }
    failure {
      slackSend(channel: "${SLACK_CHANNEL}",  message: "Build or Deploy Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER} to ${params.DEPLOY_ENV}")
    }
    always {
      cleanWs()
    }
  }
}
```


