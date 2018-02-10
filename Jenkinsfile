stage('Linux') {
  node('linux') {
    checkout scm
    sh('git submodule update --init --recursive')
    dir('libzt') {
      sh('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE -DCMAKE_EXE_LINKER_FLAGS=-pthread')
      sh('cmake --build build')
    }
    archiveArtifacts 'libzt/bin/**,libzt/include/**'
  }
}
stage('macOS') {
  node('mac') {
    checkout scm
    sh('git submodule update --init --recursive')
    dir('libzt') {
      sh('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE')
      sh('cmake --build build')
    }
    archiveArtifacts 'libzt/bin/**,libzt/include/**'
  }
}
stage('Windows') {
  node('windows') {
    checkout scm
    bat('git submodule update --init --recursive')
    dir('libzt') {
      bat('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE')
      bat('cmake --build build')
    }
    archiveArtifacts 'libzt/bin/**,libzt/include/**'
  }
}