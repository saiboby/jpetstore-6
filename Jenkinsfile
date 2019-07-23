properties([parameters([string(defaultValue: '', description: 'Specify the ImageID', name: 'ImageID')])])
	    
node{
    
    stage('Pull the code'){
        git credentialsId: 'Github', url: 'https://github.com/saiboby/jpetstore-6.git'
    }
    
    stage('Build the code'){
        def m2home=tool name: 'maven', type: 'maven'
        def mvn="${m2home}/bin/"
        sh "${mvn}/mvn package"
    }
	
    stage('junit testing'){
        junit 'target/surefire-reports/*.xml'
    }
    
    stage('Uploding the artifacts'){
        nexusArtifactUploader artifacts: [[artifactId: 'jpetstore', classifier: '', file: 'target/jpetstore.war', type: 'war']], credentialsId: 'nexus', groupId: 'jpetstore', nexusUrl: '107.23.199.186:8081/repository/jpetstore/', nexusVersion: 'nexus2', protocol: 'http', repository: 'jpetstore', version: '$BUILD_NUMBER'
    }
    
    
    stage('Deploy on tomcat'){
        sh '''rm -rf /opt/tomcat/webapps/jpetstore.*

		      cp -rf target/jpetstore.war /opt/tomcat/webapps

		      sh /opt/tomcat/bin/catalina.sh stop

		      sh /opt/tomcat/bin/catalina.sh start
      '''
    }
    
    stage('Building docker image'){
        sh 'docker build -t jpets:${BUILD_NUMBER} .'
    }
    
    stage('Validation'){
		sh '''docker images > imagelist
            cat imagelist | grep ${BUILD_NUMBER}

        if [ $? -eq 0 ]
            then
        echo "image is there"
            else
        echo "no image"
        fi
        '''
	}
	
	stage('Pushing images to Nexus Registry'){
		withCredentials([usernamePassword(credentialsId: 'nexusdocker', passwordVariable: 'password', usernameVariable: 'username'),string(credentialsId: 'nexushost', variable: 'hostname')]) {
	       		sh """docker login -u $username -p $password $hostname:8083
	                      docker tag 12cb23fcc652${params.ImageID} $hostname:8083/${params.ImageID}:$BUILD_NUMBER
                  	      docker push $hostname:8083/${params.ImageID}
                    	   """
		}
	}
		
}
