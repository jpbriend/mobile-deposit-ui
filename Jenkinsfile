def projectname='mobile-deposit'
def appname="${projectname}-ui"
def downstreamJob="../${projectname}-update-release-manifest"
if(!env.BRANCH_NAME){
    BRANCH_NAME=""
} else {
    BRANCH_NAME="/${env.BRANCH_NAME}"
}

log("setup", "BRANCH_NAME=$BRANCH_NAME")
    
node ("linux") {
    ensureMaven()
    
    def version = 0
    stage('Checkout') {
        checkout scm
        sh('git rev-parse HEAD > GIT_COMMIT')
        git_commit=readFile('GIT_COMMIT')

        def mvnHome = tool 'Maven 3.x'

        version = getResolvedVersion()
        log("Checkout", "Resolved version = ${version}")
    }

    stage('Build') {
        sh "mvn clean package -DBUILD_NUMBER=${env.BUILD_NUMBER} -DBUILD_URL=${env.BUILD_URL} -DGIT_COMMIT=${git_commit}"
    }

    stage('Test Jar') {
        //sh "java -jar target/${appname}-${version}.jar"
        sh "echo 'Pseudo Unit test execution'"
    }

    stage('Publish') {
        archive "target/${appname}-${version}.jar"
    }

    stage('Trigger Release Build') {
       build job: downstreamJob, parameters: [[$class: 'StringParameterValue', name: "app", value: "${appname}${BRANCH_NAME}"], [$class: 'StringParameterValue', name: 'revision', value: version]], wait: false
    }   
}

// ###### Functions

String getResolvedVersion() {
   ensureMaven()
   sh "mvn org.apache.maven.plugins:maven-help-plugin:2.1.1:evaluate -DBUILD_NUMBER=${env.BUILD_NUMBER} -Dexpression=project.version | grep -v '\\[' > ver.txt"
   def v = readFile 'ver.txt'
   return v.trim()
}

def ensureMaven() {
    env.PATH = "${tool 'Maven 3.x'}/bin:${env.PATH}"
}

def log (step, msg) {

    echo """************************************************************
Step: $step
$msg
************************************************************"""
}
