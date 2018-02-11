stage('Windows') {
  node('windows') {
    checkout scm
    bat('git submodule update --init --recursive')
    dir('libzt') {
      bat('''
git clean -xdf build bin_win64
set PATH=%PATH:"=%
call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\Common7\\Tools\\VsDevCmd.bat"
cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE
''')
      bat('''
set PATH=%PATH:"=%
call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\Common7\\Tools\\VsDevCmd.bat"
cmake --build build''')
      bat('move bin bin_win64')
    }
    archiveArtifacts 'libzt/bin_win64/**'
    stash includes: 'libzt/bin_win64/**', name: 'win'
  }
}
stage('Linux') {
  node('linux') {
    checkout scm
    sh('git submodule update --init --recursive')
    dir('libzt') {
      sh('git clean -xdf build bin_linux64 || true')
      sh('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE -DCMAKE_EXE_LINKER_FLAGS=-pthread')
      sh('cmake --build build')
      sh('mv bin bin_linux64')
    }
    archiveArtifacts 'libzt/bin_linux64/**,libzt/include/**'
    stash includes: 'libzt/bin_linux64/**,libzt/include/**', name: 'linux'
  }
}
stage('macOS') {
  node('mac') {
    checkout scm
    sh('git submodule update --init --recursive')
    dir('libzt') {
      sh('git clean -xdf build bin_macosuniversal || true')
      sh('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE "-DCMAKE_OSX_ARCHITECTURES=x86_64;i386"')
      sh('cmake --build build')
      sh('mv bin bin_macosuniversal')
    }
    archiveArtifacts 'libzt/bin_macosuniversal/**'
    stash includes: 'libzt/bin_macosuniversal/**', name: 'mac'
  }
}
milestone label: 'Publish', ordinal: 20
stage('Publish to GitHub') {
  node('linux') {
    withCredentials([string(credentialsId: 'HiveMP-Deploy', variable: 'GITHUB_TOKEN')]) {
      timeout(3) {
        sh('\$GITHUB_RELEASE release --user HiveMP --repo zerotier-build --tag 0.' + env.BUILD_NUMBER + ' -c ' + gitCommit + ' -n "libzt binaries (build #' + env.BUILD_NUMBER + ')" -d "This release is being created by the build server." -p')
        ws {
          sh('rm -Rf libzt || true')
          unstash 'linux'
          sh('tar -zcvf linux64-' + env.BUILD_NUMBER + '.tar.gz libzt/bin_linux64 libzt/include')
          stash includes: ('linux64-' + env.BUILD_NUMBER + '.tar.gz'), name: 'linux-archive'
        }
        ws {
          sh('rm -Rf libzt || true')
          unstash 'mac'
          sh('tar -zcvf macosuniversal-' + env.BUILD_NUMBER + '.tar.gz libzt/bin_macosuniversal libzt/include')
          stash includes: ('macosuniversal-' + env.BUILD_NUMBER + '.tar.gz'), name: 'mac-archive'
        }
        ws {
          sh('rm -Rf libzt || true')
          unstash 'win'
          sh('zip win64-' + env.BUILD_NUMBER + '.zip libzt/bin_win64 libzt/include')
          stash includes: ('win64-' + env.BUILD_NUMBER + '.zip'), name: 'win-archive'
        }
        unstash 'linux-archive'
        unstash 'mac-archive'
        unstash 'win-archive'
        sh('\$GITHUB_RELEASE upload --user HiveMP --repo zerotier-build --tag 0.' + env.BUILD_NUMBER + ' -n linux64-' + env.BUILD_NUMBER + '.tar.gz -f linux64-' + env.BUILD_NUMBER + '.tar.gz -l "libzt binaries and static libraries for Linux (64-bit)"')
        sh('\$GITHUB_RELEASE upload --user HiveMP --repo zerotier-build --tag 0.' + env.BUILD_NUMBER + ' -n macosuniversal-' + env.BUILD_NUMBER + '.tar.gz -f macosuniversal-' + env.BUILD_NUMBER + '.tar.gz -l "libzt binaries and static libraries for macOS (Universal)"')
        sh('\$GITHUB_RELEASE upload --user HiveMP --repo zerotier-build --tag 0.' + env.BUILD_NUMBER + ' -n win64-' + env.BUILD_NUMBER + '.zip -f win64-' + env.BUILD_NUMBER + '.zip -l "libzt binaries and static libraries for Windows (64-bit)"')
        sh('\$GITHUB_RELEASE edit --user HiveMP --repo zerotier-build --tag 0.' + env.BUILD_NUMBER + ' -n "libzt binaries (build #' + env.BUILD_NUMBER + ')" -d "These are automatically built binaries and static libraries for libzt (ZeroTier). These binaries are GPL-licensed unless you have a commercial license from ZeroTier, Inc. See the README in this repository for more information."')
      }
    }
  }
}