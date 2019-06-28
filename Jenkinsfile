def git_repo = 'http://gogs-cimb-infra.apps.cimb.rht-labs.com/muttaqin/esb-mini.git'
def git_branch = 'master'
def nexus_repo = 'http://nexus-cimb-infra.apps.cimb.rht-labs.com/repository/cimb-niaga-apps/'


node ('maven'){
    stage('Checkout') {
        git url: "$git_repo", branch: "$git_branch"
    }
    stage ('Versioning'){
        pomVersion = getVersionFromPom('pom.xml')
        appVersion = pomVersion.substring(0, pomVersion.lastIndexOf('.')) + ".$BUILD_NUMBER"
        sh "sed -i 's;<version>$pomVersion</version>;<version>$appVersion</version>;' pom.xml"
    }
    stage('Build') {
        

        withCredentials([[$class: 'UsernamePasswordMultiBinding', 
            credentialsId: 'bfsIntNexusJenkinsCredential',
            usernameVariable: 'nexus_username', passwordVariable: 'nexus_password']]) {
                prepareSettingsXml(nexus_username, nexus_password)
        }
        
        sh 'mvn -s settings.xml clean install -D skipTests'
    }
    stage ('Test'){
        sh 'mvn -s settings.xml test'
    }
    stage ('Archive'){
        addDistributionToPom('pom.xml', nexus_repo)
        sh 'mvn -s settings.xml deploy'
    }
}


def getVersionFromPom(pom) {
  def matcher = readFile(pom) =~ '<version>(.+)</version>'
  matcher ? matcher[0][1] : null
}

def getGroupIdFromPom(pom) {
  def matcher = readFile(pom) =~ '<groupId>(.+)</groupId>'
  matcher ? matcher[0][1] : null
}

def getArtifactIdFromPom(pom) {
  def matcher = readFile(pom) =~ '<artifactId>(.+)</artifactId>'
  matcher ? matcher[0][1] : null
}

def getReleaseFromMetaData(pom) {
  def matcher = readFile(pom) =~ '<release>(.+)</release>'
  matcher ? matcher[0][1] : null
}

def addDistributionToPom(pom, nexus_repo) {
   suffix = """<distributionManagement>
  <repository>
      <id>internal.repo</id>
      <name>Internal Repo</name>
      <url>$nexus_repo</url>
  </repository>
  <snapshotRepository>
      <id>internal.repo</id>
      <name>Internal Repo</name>
      <url>$nexus_repo</url>
  </snapshotRepository>
</distributionManagement></project>"""

   content = readFile(pom)
   newContent = content.substring(0, content.lastIndexOf('</project>')) + suffix
   writeFile file: pom, text: newContent
}


def prepareSettingsXml(nexus_username, nexus_password) {
    settings = """<?xml version=\"1.0\" encoding=\"UTF-8\"?>
    <settings xmlns=\"http://maven.apache.org/SETTINGS/1.0.0\"
            xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"
            xsi:schemaLocation=\"http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd\">
    <mirrors>
        <mirror>
        <id>nexus</id>
        <mirrorOf>*</mirrorOf>
        <url>http://nexus-cimb-infra.apps.cimb.rht-labs.com/repository/maven-public/</url>
        </mirror>
    </mirrors>
    <profiles>
        <profile>
        <id>nexus</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <repositories>
            <repository>
            <id>central</id>
            <url>http://central</url>
            <releases><enabled>true</enabled></releases>
            <snapshots><enabled>true</enabled></snapshots>
            </repository>
        </repositories>
        <pluginRepositories>
            <pluginRepository>
            <id>central</id>
            <url>http://central</url>
            <releases><enabled>true</enabled></releases>
            <snapshots><enabled>true</enabled></snapshots>
            </pluginRepository>
        </pluginRepositories>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>nexus</activeProfile>
    </activeProfiles>
    <servers>
        <server>
            <id>internal.repo</id>
            <username>$nexus_username</username>
            <password>$nexus_password</password>
        </server>
    </servers>
    </settings>"""

    writeFile file: 'settings.xml', text: settings
}