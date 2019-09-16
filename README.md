# BackBaseTest

Example for other teams pipeline

stage("Run main job") {
    steps {
        script {
            build(job: 'buildApplication', parameters: [
                    string(name: 'REPOSITORY', value: "test"),
                    string(name: 'BRANCH', value: "master"),
            ], wait: false)
        }

    }
}
