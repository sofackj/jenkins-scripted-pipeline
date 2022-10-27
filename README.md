# Jenkins Pipelines (v2.361.2)

## Scripted Pipelines

### Variables in Scripted pipelines
Check his [guide](https://e.printstacktrace.blog/jenkins-pipeline-environment-variables-the-definitive-guide/)
- Simple way
```sh
// Define the variable without env prefix
def MY_VAR = 'Hello World' // def is not necessary but for readibility you need to mention it
// Start job
node() {
    stage('Using env prefix') {
        echo MY_VAR // SUCCESS
        echo "${MY_VAR}" // SUCCESS
        echo '${MY_VAR}' // FAIL -> output : ${MY_VAR}
        sh 'echo ${MY_VAR}' // FAIL -> no output
        sh "echo ${MY_VAR}" // SUCCESS
    }
}
```
- Using the env prefix
```sh
// Define the variable
env.MY_VAR = 'Hello World'
// Start job
node() {
    stage('Using env prefix') {
        echo MY_VAR // SUCCESS
        echo "${MY_VAR}" // SUCCESS
        echo '${MY_VAR}' // FAIL
        sh 'echo ${MY_VAR}' // SUCCESS
        sh "echo ${MY_VAR}" // SUCCESS
    }
}
```
- Define variables : Dynamic way
```sh
// Define the variable
def ENV_VAR = 'MY_VAR' 
env.setProperty(ENV_VAR, 'Hello World')
// Start job
node() {
    stage('Using dynamic declaration') {
        echo MY_VAR // SUCCESS
        echo "${MY_VAR}" // SUCCESS
        echo '${MY_VAR}' // FAIL -> output : ${MY_VAR}
        sh 'echo ${MY_VAR}' // SUCCESS
        sh "echo ${MY_VAR}" // SUCCESS
    }
}
```
- Define variables using [withEnv](https://www.cloudbees.com/blog/managing-your-jenkins-environment-using-withenv-a-tutorial)

```sh
// Define the variable
withEnv(['MY_VAR=Hello World']) {
    // Start the job
    node(){
        stage('Using withEnv method') {
        echo MY_VAR // SUCCESS
        echo "${MY_VAR}" // SUCCESS
        echo '${MY_VAR}' // FAIL -> output : ${MY_VAR}
        sh 'echo ${MY_VAR}' // SUCCESS
        sh "echo ${MY_VAR}" // SUCCESS
        }
    }
}
```
### Lists and dictionaries in pipelines
#### Declaration
In the following examples, the method with env be used, but outputs for the simple way declaration will be shown
- Declaration of a list type variable
```sh
// Define the variable
env.MY_VAR = ['A', 'B', 'C']
// def MY_VAR = ['A', 'B', 'C'] also works
// Start job
node() {
    stage('Using env prefix') {
        echo MY_VAR // SUCCESS
        // (without env.) FAIL -> exit the pipeline
        echo "${MY_VAR}" // SUCCESS
        sh 'echo ${MY_VAR}' // SUCCESS
        // (without env.) FAIL -> no output
        sh "echo ${MY_VAR}" // SUCCESS
    }
}
```
It also works using the dynamic way :
```sh
// Define the variable
def ENV_VAR = 'MY_VAR' 
env.setProperty(ENV_VAR, ['A', 'B', 'C'])
```
No declaration was a success with the "withENd" method
- Declaration of a dictionary type variable
```sh
// Define the variable
// It also Works with def MY_VAR =...
env.MY_VAR = [
    'A' : 'Hello',
    'B' : 'bye',
    'C' : 'nice',
    'D' : 'ciao'
    ]
```
As for the list section, same results are obtained
#### Properties for lists and dictionaries
[To complete]

### Assign a command output to a variable
Don't hesitate to check the [documenation](https://www.jenkins.io/doc/pipeline/steps/workflow-durable-task-step/#sh-shell-script)
- Assign the output of the command to the variable
```sh
node(){
    // The command output 'ls -la' is assigned to the variable LIST_FILE
    def LIST_FILES = sh (
        // Definition of the command
        script: 'ls -la',
        // Return the display of the command to the variable
        returnStdout: true
        ).trim()// No line jump at the end
        .readLines()// Generate a list and each line is an element of the list
        .drop(0)// Remove the 0 first elements of the list
        .join(" ")// Join the elements of the list with " "
    
    // Display the variable with 'Hello World' around
    echo "Hello ${LIST_FILES} World"
}
```
- Assign the status of the command to the variable
```sh
node(){
    // The command output 'ls -la' is assigned to the variable LIST_FILE
    def STATUS_COM = sh (
        // Definition of the command
        script: 'cat this-file-doesnt-exist.txt 2>&1',
        // Return the status of the command to the variable
        returnStatus: true
        )
        
    // Display the variable with 'Hello World' around
    echo "Status of the command : ${STATUS_COM} "
}
```

### How to do loops in pipelines
Thanks to [Oiflnd](https://gist.github.com/oifland/ab56226d5f0375103141b5fbd7807398)

- Loops on a list without 'for'
```sh
// List type variables doesn't work with synamic and static declaration
def MY_LIST = ['a', 'b', 'c']
// Start job
node() {
    stage('Loop on the abcs variable') {
        MY_LIST.each { item ->
        sh "echo This is ${item}"
        }
    }
}
```
- Loops on list with 'for'
```sh
//List type variables doesn't work with synamic and static declaration
MY_LIST = ['a', 'b', 'c']
// Start job
node() {
    stage('Loop on the abcs variable') {
        echo "Here the elements of the list"
        for (int i = 0; i < MY_LIST.size(); i++) {
        sh "echo Hello ${my_list[i]}"
        }
    }
}
```
### Pipelines with conditions
```sh
// Define a variable
env.MY_VAR = 'my-value0'
// Start the job
node {
    stage('Test conditions'){
        if (env.MY_VAR == "my-value0") {
            echo "Test SUCCEEDED"
            // Other steps can be added
        } else {
            echo "Test FAILED"
            // Other steps can be added
        }
    }
}
```

### Try/Catch with pipelines
```sh
node {
    stage('Try and catch') {
        // Command(s) to test
        try {
            // Failed on purpose
            sh 'cat this-file-doesnt-exist.txt'
            // Answer to the failure
        } catch (Exception e) {
            // "catch (exc)" can also be use but no error type will be raised
            // Raise the error type
            echo e.toString()
            // Output -> hudson.AbortException: script returned exit code 1
            echo 'Something goes wrong !'
        }
    }
}
```
### Steps in parallel
- Method 1 : Simple way
```sh
node {
    stage('Build')
    {
        println 'I prepare the build for the parallel steps'
    }
    
    stage('Test') {
        parallel
        (
            "Step 1" :
            {
                stage("checkout") {echo "Hello World"};
                stage("build") {echo "Hello World"};
                stage("test") {echo "Hello World"}
            },
            "Step 2" :
            {
                stage("checkout") {echo "Hello World"};
                stage("build") {echo "Hello World"};
                stage("test") {echo "Hello World"}
            },
            "Step 3" :
            {
                stage("checkout") {echo "Hello World"};
                stage("build") {echo "Hello World"};
                stage("test") {echo "Hello World"}
            }
        )
    }
}
```
- Method 2 : Using a variable
````sh
// Declare an empty dictionary
def test = [:]
// Fill the dictionary with the following block
// First block
test["first-task"] = {
    stage ("first-task") {
        stage ("first-task-step-1") {
            sh "echo Hello World"
        }
        stage ("first-task-step-2") {
            sh "echo Eveything is fine"
        }
    }
}
// Second block
test["second-task"] = {
    stage ("second-task") {
        stage ("second-task-step-1") {
            sh "echo Hello World"
        }
        stage ("second-task-step-2") {
            sh "echo Eveything is fine"
        }
    }
}
// Job start
node {
    stage ('Processes in parallel'){
        // Setup the different tasks in parallel
        parallel test
    }
    // Add an other stage after parallel processes
    stage ('End stage') {
       sh "echo Everything is done..."
   }
}
```
### Examples
- Check the ping of several servers
```sh
// Define the variable
def IP_NODES = [
    // Specs for node ansible
    [
        'name' : 'ansible',
        'ipv4' : '192.168.73.128'
        ],
    // Specs for node jenkins
    [
        'name' : 'jenkins',
        'ipv4' : '192.168.73.129'
        ],
    // Specs for node docker
    [
        'name' : 'docker',
        'ipv4' : '192.168.73.130'
        ]
    ]
// Start job and choose the ansible-controller as node
node('ansible-controller') {
    stage('Check SSH connection for nodes') {
        // Start the loop for all nodes
        // Each node is defined by the variable 'node'
        IP_NODES.each{ node ->
        echo "${node['name']} : ${node['ipv4']}"
        echo "Test ping node ${node['name']}"
        // Try and catch for the ping command
        try {
            sh "ping -c 1 ${node['ipv4']}"
        // If no ping, going to catch and fails the pipeline
        } catch (exc) {
            echo "FAILED !!! ${node['name']} is out of reach..."
            sh "exit 1"
        // If ping detected, display the following message
        } finally {
            echo "SUCCESS !!! ${node['name']} is responding !"
        }
        }
    }
}
```
- Check a command and exit the pipeline if necessary
```sh
// Command to test
def COMMAND = 'cat this-file-doesnt-exist.txt'
// Path to the output file
def OUTPUT_FILE = 'output.txt'
// Job start
node(){
    // The command output 'ls -la' is assigned to the variable LIST_FILE
    def STATUS_COM = sh (
        // Definition of the command
        script: "${COMMAND} >${OUTPUT_FILE} 2>&1",
        // Return the status of the command to the variable
        returnStatus: true
        )
    // Check the status of the command
    if (STATUS_COM == 0){
        // If the command is a success, display this message
        echo "SUCCESS"
    } else {
        // Display the content of the file when this is an error
        echo "ERROR MSG => ${readFile(OUTPUT_FILE).trim()}"
        sh "exit 1"// In case you want to break the pipeline
    }
}
```
- Delete some directories for clean up
```sh
// List of directories
def DIR_LIST = ['dir1', 'dir2', 'dir3']
// STart job
node("ansible-controller"){
    for (directory in DIR_LIST){
        dir(directory){
            deleteDir()
        }
    }
}
```
- Start Httpd containers in parallel
```sh
// Job on the node with the label 'docker'
node('docker') {
    stage('Start containers') {
        parallel(
            // First step in parallel
            'port-8081':
            {
                // Run a first httpd container
                docker.image('httpd:alpine')
                .withRun(' --name my-container -p 8081:80')
                {
                    sh 'curl http://localhost:8081'
                    sleep 5
                }
            },
            // Second step in parallel
            'port-8082':
            {
                // Run a second httpd container
                docker.image('httpd:alpine')
                .withRun(' --name my-container-2 -p 8082:80')
                {
                    sh 'curl http://localhost:8082'
                    sleep 5
                }
            },
            // Third step in parallel
            'Containers list' : 
            {
                // List active containers after 10s
                sh 'sleep 5;docker ps'
            }
        )
    }
}
```
