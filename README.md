# Jenkins Pipelines (v2.361.2)

## Scripted Pipelines

### Variables in Scripted pipelines
Check his [guide](https://e.printstacktrace.blog/jenkins-pipeline-environment-variables-the-definitive-guide/)
- Define variables : Simple way
```sh
node {
    stage('Simple way'){
        env.MY_VAR = 'my-value1'
    }
    stage('Dynamic way'){
        def envVarName = 'MY_SEC_VAR' 
        env.setProperty(envVarName, 'my-value2')
    }
    stage('Print MY_VAR'){
        sh 'echo ${MY_VAR}'
        sh 'echo ${MY_SEC_VAR}'
    }
}
```
- Define variables using [withEnv](https://www.cloudbees.com/blog/managing-your-jenkins-environment-using-withenv-a-tutorial)

```sh
node {
      withEnv(['value=World']) {
           stage('Test variable') {
               sh 'echo Hello, ${value}'
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
