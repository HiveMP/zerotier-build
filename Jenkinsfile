stage('Linux') {
  node('linux') {
    checkout scm
    sh('git submodule update --init --recursive')
    dir('libzt') {
      sh('git clean -xdf build || true')
      sh('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE -DCMAKE_EXE_LINKER_FLAGS=-pthread')
      sh('cmake --build build')
      sh('mv bin bin_linux')
    }
    archiveArtifacts 'libzt/bin_linux/**,libzt/include/**'
  }
}
stage('macOS') {
  node('mac') {
    checkout scm
    sh('git submodule update --init --recursive')
    dir('libzt') {
      sh('git clean -xdf build || true')
      sh('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE')
      sh('cmake --build build')
      sh('mv bin bin_mac')
    }
    archiveArtifacts 'libzt/bin_mac/**'
  }
}
stage('Windows') {
  node('windows') {
    checkout scm
    bat('git submodule update --init --recursive')
    dir('libzt') {
      bat('git clean -xdf build')
      bat('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE -G "MSYS Makefiles"')
      bat('cmake --build build')
      bat('move bin bin_win')
    }
    archiveArtifacts 'libzt/bin_win/**'
  }
}