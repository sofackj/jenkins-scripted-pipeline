# Jenkins Pipelines

## Scripted Pipelines

### Variables in Scripted pipelines
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
```sh
node {
      withEnv(['value=World']) {
           stage('Test variable') {
               sh 'echo Hello, ${value}'
           }
      }
  }
```
