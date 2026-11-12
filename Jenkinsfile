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
    }
}
