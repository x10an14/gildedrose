node {
   stage('Preparation') { // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/meekrosoft/gildedrose.git'
      script {
          def pomFile = readMavenPom(file: 'pom.xml')
          if (pomFile.version.endsWith('-SNAPSHOT')) {
              def currentDateTime = new Date()
              String timeStamp = currentDateTime.format("yyyy.MM.dd'T'HH.mm.ss.S")
              String commitSha1 = sh(returnStdout: true, script: 'git log -n 1 --pretty=%H').trim()
              pomFile.version = "${pomFile.version.replace('-SNAPSHOT', '')}-${timeStamp}-${commitSha1}"
              writeMavenPom(file: 'pom.xml', model: pomFile)
          }
      }
   }
   stage('Build') {
     sh 'docker run -i --rm --name my-maven-project -v ~/.m2:/root/.m2  -v "$PWD":/usr/src/mymaven -w /usr/src/mymaven maven:3-jdk-8 mvn install'
   }
   stage('Results') {
      junit '**/target/surefire-reports/TEST-*.xml'
      archive 'target/*.jar'
   }
   stage('Javadoc') {
      sh 'docker run -i --rm --name my-maven-project -v ~/.m2:/root/.m2  -v "$PWD":/usr/src/mymaven -w /usr/src/mymaven maven:3-jdk-8 mvn site'
      archive 'target/site/**/*'
   }
   stage('Deploy') {
      sh 'docker run -i --rm --name my-maven-project -v ~/.m2:/root/.m2  -v "$PWD":/usr/src/mymaven -w /usr/src/mymaven maven:3-jdk-8 mvn deploy'
      archive 'target/site/**/*'
   }
}
