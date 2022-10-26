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
    'C' : 'Nice'
    ]
```
As for the list section, sma results are obtained

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

### How to do loops in pipelines
Thanks to [Oiflnd](https://gist.github.com/oifland/ab56226d5f0375103141b5fbd7807398)

- Loops on a list without 'for'
```sh
// List type variables doesn't work with synamic and static declaration
my_list = ['a', 'b', 'c']
// Start job
node() {
    stage('Loop on the abcs variable') {
        my_list.each { item ->
        sh "echo This is ${item}"
        }
    }
}
```
- Loops on list with 'for'
```sh
//List type variables doesn't work with synamic and static declaration
my_list = ['a', 'b', 'c']
// Start job
node() {
    stage('Loop on the abcs variable') {
        echo "Here the elements of the list"
        for (int i = 0; i < my_list.size(); i++) {
        sh "echo Hello ${my_list[i]}"
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
