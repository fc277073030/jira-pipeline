def issue = jiraGetIssue idOrKey: issue_key, site: 'jira'
def filename = issue.data.fields.attachment[0].filename
def dbInstance = issue.data.fields.customfield_10218.value + '_' + issue.data.fields.customfield_10219.value
def databaseName = issue.data.fields.customfield_10302

pipeline {
    agent any

    //parameters {
     //   string(name: 'filename', defaultValue: filename, description: 'Who should I say hello to?')
     //   string(name: 'dbInstance', defaultValue: dbInstance, description: 'db instance')
     //   string(name: 'databaseName', defaultValue: databaseName, description: 'database name')
   // }

    stages {
        stage('JIRA') {
            steps {
                script {
                    def id = issue.data.fields.attachment[0].id
                    jiraDownloadAttachment id: id, file: filename, override: true, site: 'jira'
                    
                    echo 'jira问题id是:' + ' ' + id
                    echo '附件名称为:' + ' ' + filename
                    echo 'dbInstance:' + ' ' + dbInstance
                    echo 'databaseName:' + ' ' + databaseName
                }
            }
        }

        stage('UploadToMinio') {
            steps {
                minio bucket: 'jenkins-upload',
                    credentialsId: 'minio',
                    excludes: '',
                    host: 'http://192.168.5.151:9000',
                    includes: filename,
                    targetFolder: ''
            }
        }

        stage('AnsibleTower') {
            steps {
                ansibleTower(
                    towerServer: 'ansible-awx',
                    towerCredentialsId: 'ansible-awx',
                    templateType: 'job',
                    jobTemplate: 'Mysql Template',
                    towerLogLevel: 'full',
                    inventory: 'Mysql Inventory',
                    jobTags: '',
                    skipJobTags: '',
                    limit: '',
                    removeColor: false,
                    verbose: true,
                    credential: '',
                    extraVars: """---
                        filename: "$filename"
                        db_instance: "${dbInstance}"
                        backup_level: "database"
                        database: "${databaseName}"
                        """,
                    async: false
                )
            }
        }
    }

    post {
        always {
            echo 'One way or another, I have finished'
            deleteDir() /* clean up our workspace */
        }
        success {
            echo 'I succeeded!'
            script {
                def transitionSuccess = '971'
                def transitionInput =
                [
                    transition: [
                        id: transitionSuccess
                    ]
                ]

                jiraTransitionIssue idOrKey: "${issue_key}", input: transitionInput, site: 'jira'
            }
        }

        failure {
            echo 'I failed!'
            script {
                def transitionFail = '981'
                def transitionInput =
                [
                    transition: [
                        id: transitionFail
                    ]
                ]

                jiraTransitionIssue idOrKey: "${issue_key}", input: transitionInput, site: 'jira'
            }
        }
        changed {
            echo 'Things were different before...'
        }
    }
}
