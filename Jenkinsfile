pipeline {
  environment {
    //
    // Initial variable setup
    //
    // you need a credential named 'docker-hub' with your DockerID/password to push images
    // we will create DOCKER_HUB_USR and DOCKER_HUB_PSW from the docker-hub credential
    CREDENTIAL = "docker-hub"
    DOCKER_HUB = credentials("$CREDENTIAL")
    //
    // now we'll set up our image name/tag
    //
    REGISTRY = "docker.io"
    REPOSITORY = "${DOCKER_HUB_USR}/${JOB_BASE_NAME}"
    TAG = "${BRANCH_NAME}-anchorectl"
    IMAGE = "${REGISTRY}/${REPOSITORY}:${TAG}"
    BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
    //
    // and we need credentials for anchorectl
    //
    ANCHORECTL_URL = credentials('Anchorectl_Url')
    ANCHORECTL_USERNAME = credentials('Anchorectl_Username')
    ANCHORECTL_PASSWORD = credentials('Anchorectl_Password')
    //
    // if you want to gate on policy failures, set this to "true"
    //
    ANCHORE_FAIL_ON_POLICY = "false"
    //
  } // end environment 
  
  agent any
  
  stages {
    
    stage('Checkout SCM') {
      steps {
        checkout scm
      } // end steps
    } // end stage "checkout scm"
    
    stage('Verify Tools') {
      steps {
        sh """
          which docker
          ###
          ### could additionally check for anchorectl here if you didn't want to fresh  
          ### install it every time down below in the analyze stage
          """
      } // end steps
    } // end stage "Verify Tools"

    
    stage('Build and Push Image') {
      steps {
        sh """
          echo ${DOCKER_HUB_PSW} | docker login -u ${DOCKER_HUB_USR} --password-stdin
          docker build -t ${IMAGE} --pull -f ./Dockerfile .
          docker push ${IMAGE}
        """
      } // end steps
    } // end stage "build and push"
    
    stage('Analyze Image with anchorectl') {
      steps {
        //
        // first, install latest version of anchorectl 
        // (you could just install this into the jenkins build node at /usr/local/bin but for demo 
        // purposes this is probably better since it ensures I always have the newest version and I 
        // don't run this pipeline very frequently)
        //
        sh """
          curl -sSfL  https://anchorectl-releases.anchore.io/anchorectl/install.sh  | sh -s -- -b $HOME/.local/bin v5.0.0
          export PATH="$HOME/.local/bin/:$PATH"   
          ### you could also install grype and syft depending on what you need to do:
          # curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b $HOME/.local/bin 
          # curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b $HOME/.local/bin 
          ### if you want to debug and check connectivity and anchorectl variables etc, 
          # anchorectl system status
        """
        // 
        // now we queue the image for analysis
        // the --wait is only necessary if you want to check the results, you can omit that if you are just
        // queueing the image and will check results later.
        //
        sh """
          ### you almost always should use --force when supplying a dockerfile
          ${HOME}/.local/bin/anchorectl image add --no-auto-subscribe --wait --force --dockerfile ./Dockerfile --from registry --annotation build_tool=jenkins --annotation scan_type=distributed ${IMAGE}
          #
          ### the jenkins plugin will pull the evaluation and vulnerability output and 
          ### archive them as build artifacts, if you want to do that here, use these:
          ${HOME}/.local/bin/anchorectl image vuln ${IMAGE}
          ${HOME}/.local/bin/anchorectl image check --detail ${IMAGE}
          #
          ### alternatively, if you want to break the pipeline if the policy evaluation fails,
          #
          # set -o pipefail
          # if [ "$ANCHORE_FAIL_ON_POLICY" == "true" ] ; then 
          #   anchorectl image check --detail --fail-based-on-results ${IMAGE} ; 
          # else 
          #   anchorectl image check --detail ${IMAGE} ; 
          # fi    
          #
        """
        //
        // if you want continuous re-evaluation in the background, you can turn it on with these:
        //   anchorectl subscription activate policy_eval ${IMAGE}
        //   anchorectl subscription activate vuln_update ${IMAGE}
        // and in this case you would probably also want to configure "policy & vulnerability" updates
        // in "Events & Notifications" -> "Manage Notification Endpoints" 
        //
      } // end steps
    } // end stage "analyze image with anchorectl"     
    
    // optional, you could promote the image here 
    stage('Promote Image') {
      steps {
        sh """
          ### retag the image as "prod"
          docker tag ${IMAGE} ${IMAGE}-prod
          docker push ${IMAGE}-prod
        """
      } // end steps
    } // end stage "Promote Image"        
        
  } // end stages

  post {
    always {
      //
      // don't need the image(s) anymore so let's rm it
      //
      sh 'docker image rm ${IMAGE} ${IMAGE}-prod || failure=1'
      // the || failure=1 just allows us to continue even if one or both of the tags we're
      // rm'ing doesn't exist (e.g. if the evaluation failed, we might end up here without 
      // re-tagging the image, so ${BRANCH_NAME} wouldn't exist.
      //
    } // end always
  } //end post
  
} // end pipeline 
