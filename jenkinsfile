node {
    agent {
        label 'master'
    }
    def PRS = []
    stage('QueryGithubPR') {
        skipDefaultCheckout()
        // if project is private, add authrization header here.
        pullreqsjson = sh(returnStdout: true, script: 'curl https://api.github.com/repos/kokoisN1/MyJenkins/pulls').trim()
        PRS = get_open_PR(pullreqsjson)
        if (PRS != []) {
            println "new/open pull/merge reuests found: " + PRS
            deleteDir()
            checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'kokoisN1', url: 'https://github.com/kokoisN1/MyJenkins.git']]])
        } else {
            println "No new/open pull/merge reuests found."
        }
    }
    stage('RunPythonScript') {
        if (PRS == []) {
            println "No new/open PR/merge requests found, not ruuning python script."
        } else {
            sh "docker run --rm -v /var/jenkins_home:/var/jenkins_home  --name my-running-script python:3 python $WORKSPACE/main.py"
        }
    }
} // node

def get_open_PR(String json_to_search){
    def ret = []
    String to_find = 'title'
    String to_find_state = 'state'
    int index = json_to_search.indexOf(to_find)
    int state_index = json_to_search.indexOf(to_find_state)
    while (index > -1) {
        pair = [:]
        // the "4" below is due to the '": "' string to jump between the JSON 
        // key and value fields.
        start_of_value = index + 4 + to_find.length()
        end_of_value = json_to_search.indexOf("\"", start_of_value)
        PR_name = json_to_search.substring(start_of_value, end_of_value)

        start_of_state_value = state_index + 4 + to_find.length()
        end_of_state_value = json_to_search.indexOf("\"", start_of_state_value)
        PR_state = json_to_search.substring(start_of_state_value, end_of_state_value)

        if (PR_state == 'open') {
            ret << PR_name
        }
        index  = json_to_search.indexOf(to_find, index + to_find.length())
        state_index  = json_to_search.indexOf(to_find_state, state_index + to_find_state.length())
    }
    return ret 
} // get_open_PR()
