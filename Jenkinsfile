pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git submodule update --init --recursive'
            }
        }

        stage('Verify Source') {
            steps {
                sh '''
                    echo "Checking Hugo project files..."
                    test -f hugo.toml
                    test -f content/posts/welcome.md
                    test -f themes/PaperMod/theme.toml || test -f themes/PaperMod/hugo.toml
                    echo "Hugo source files verified."
                '''
            }
        }

        stage('Build Hugo Site') {
            steps {
                sh '''
                    echo "Building Hugo website..."
                    hugo --minify
                    test -f public/index.html
                    echo "Hugo website built successfully."
                '''
            }
        }
	stage('Deploy Hugo Site') {
	    steps {
		sh '''
		    set -eu

		    echo "Preparing Hugo deployment..."

		    test -f public/index.html
		    test -d /var/jenkins_deploy/site

		    echo "Synchronizing website files..."

		    rsync -av --delete \
			public/ \
			/var/jenkins_deploy/site/

		    echo "Hugo website deployed successfully."
		'''
	    }
	}
    }
}
