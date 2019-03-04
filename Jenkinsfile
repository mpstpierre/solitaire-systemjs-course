stage 'CI'
node {

	checkout scm
    //git branch: 'jenkins2-course', 
    //    url: 'https://github.com/mpstpierre/solitaire-systemjs-course'

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    bat 'npm --proxy http://www-proxy.lmig.com:80 install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'
    
    // test with PhantomJS for "fast" "generic" results
    // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
    bat 'npm --proxy http://www-proxy.lmig.com:80 run test-single-run -- --browsers PhantomJS'
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', 
          testResults: 'test-results/**/test-results.xml'])
          
}

node ('windows') {
    // on windows use: bat 'dir'
    bat 'dir'

    // on windows use: bat 'del /S /Q *'
    bat 'del /S /Q *'

    unstash 'everything'

    // on windows use: bat 'dir'
    bat 'dir'
}

parallel chrome: {
    runTests("Chrome")
}, firefox: {
    runTests("Firefox")
}

def runTests(browser) {
    node {
        // on windows use: bat 'del /S /Q *'
        bat 'del /S /Q *'

        unstash 'everything'

        // on windows use: bat "npm run test-single-run -- --browsers ${browser}"
        bat "npm --proxy http://www-proxy.lmig.com:80 run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver', 
              testResults: 'test-results/**/test-results.xml'])
    }
}

node {
    notify("Deploy to staging?")
}

input 'Deploy to staging?'

stage name: 'Deploy', concurrency: 1
node {
    // write build number to index page so we can see this update
    
    // deploy to a docker container mapped to port 3000
    // on windows use: bat 'docker-compose up -d --build'
    //bat 'docker-compose up -d --build'
    
    notify 'Solitaire Deployed!'
}

def notify(status){
    emailext (
      to: "marc.stpierre@lmi.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
