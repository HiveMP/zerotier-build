def gitCommit = ""
stage('Build') {
  def parallelMap = [:]
  parallelMap["Linux"] = {
    node('linux') {
      withCredentials([string(credentialsId: 'HiveMP-Deploy', variable: 'GITHUB_TOKEN')]) {
        // Try to load credential so we know they'll work at the end of the script.
      }
      gitCommit = checkout(poll: true, changelog: true, scm: scm).GIT_COMMIT
      sh('git submodule update --init --recursive')
      dir('libzt') {
        sh('git clean -xdf build bin_linux64 || true')
        sh('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE -DCMAKE_EXE_LINKER_FLAGS=-pthread')
        sh('cmake --build build')
        sh('mv bin bin_linux64')
      }
      archiveArtifacts 'libzt/bin_linux64/**,libzt/include/libzt.h,libzt/include/libztDebug.h,libzt/include/libztDefs.h,libzt/include/lwipopts.h,libzt/ext/lwip/src/include/lwip/**,libzt/ext/lwip-contrib/ports/unix/include/**'
      stash includes: 'libzt/bin_linux64/**,libzt/include/libzt.h,libzt/include/libztDebug.h,libzt/include/libztDefs.h,libzt/include/lwipopts.h,libzt/ext/lwip/src/include/lwip/**,libzt/ext/lwip-contrib/ports/unix/include/**', name: 'linux'
    }
  }
  /*
  parallelMap["macOS"] = {
    node('mac') {
      checkout(poll: false, changelog: false, scm: scm)
      sh('git submodule update --init --recursive')
      dir('libzt') {
        sh('git clean -xdf build bin_macosuniversal || true')
        sh('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE "-DCMAKE_OSX_ARCHITECTURES=x86_64;i386"')
        sh('cmake --build build')
        sh('mv bin bin_macosuniversal')
      }
      archiveArtifacts 'libzt/bin_macosuniversal/**'
      stash includes: 'libzt/bin_macosuniversal/**,libzt/include/libzt.h,libzt/include/libztDebug.h,libzt/include/libztDefs.h,libzt/include/lwipopts.h,libzt/ext/lwip/src/include/lwip/**,libzt/ext/lwip-contrib/ports/unix/include/**', name: 'mac'
    }
  }
  */
  parallelMap["Windows x64"] = {
    node('windows') {
      checkout(poll: false, changelog: false, scm: scm)
      bat('git submodule update --init --recursive')
      dir('libzt') {
        bat('''
  git clean -xdf build bin_win64
  git checkout -f CMakeLists.txt
  ''')
        powershell('''
  $Value = Get-Content -Raw CMakeLists.txt
  $Value = $Value.Replace("\\\\x86", "\\\\x64")
  $Value = $Value.Replace("/x86", "/x64")
  Set-Content -Path CMakeLists.txt -Value $Value
  ''')
        bat('''
  set PATH=%PATH:"=%
  call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
  cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE -G "Visual Studio 15 2017 Win64"
  ''')
        bat('''
  set PATH=%PATH:"=%
  call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars64.bat"
  cmake --build build''')
        bat('move bin bin_win64')
      }
      archiveArtifacts 'libzt/bin_win64/**,libzt/ext/lwip-contrib/ports/win32/include/**'
      stash includes: 'libzt/bin_win64/**,libzt/include/libzt.h,libzt/include/libztDebug.h,libzt/include/libztDefs.h,libzt/include/lwipopts.h,libzt/ext/lwip/src/include/lwip/**,libzt/ext/lwip-contrib/ports/win32/include/**', name: 'win64'
    }
  }
  parallelMap["Windows x86"] = {
    node('windows') {
      checkout(poll: false, changelog: false, scm: scm)
      bat('git submodule update --init --recursive')
      dir('libzt') {
        bat('''
  git clean -xdf build bin_win32
  git checkout -f CMakeLists.txt
  set PATH=%PATH:"=%
  call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars32.bat"
  cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE -G "Visual Studio 15 2017"
  ''')
        bat('''
  set PATH=%PATH:"=%
  call "C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Auxiliary\\Build\\vcvars32.bat"
  cmake --build build''')
        bat('move bin bin_win32')
      }
      archiveArtifacts 'libzt/bin_win32/**'
      stash includes: 'libzt/bin_win32/**,libzt/include/libzt.h,libzt/include/libztDebug.h,libzt/include/libztDefs.h,libzt/include/lwipopts.h,libzt/ext/lwip/src/include/lwip/**,libzt/ext/lwip-contrib/ports/win32/include/**', name: 'win32'
    }
  }
  parallel (parallelMap)
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
          sh('mv libzt/ext/lwip/src/include/lwip libzt/include/')
          sh('mv libzt/ext/lwip-contrib/ports/unix/include/* libzt/include/lwip/')
          sh('tar -zcvf linux64-' + env.BUILD_NUMBER + '.tar.gz libzt/bin_linux64 libzt/include')
          stash includes: ('linux64-' + env.BUILD_NUMBER + '.tar.gz'), name: 'linux-archive'
        }
        /*ws {
          sh('rm -Rf libzt || true')
          unstash 'mac'
          sh('mv libzt/ext/lwip/src/include/lwip libzt/include/')
          sh('mv libzt/ext/lwip-contrib/ports/unix/include/* libzt/include/lwip/')
          sh('tar -zcvf macosuniversal-' + env.BUILD_NUMBER + '.tar.gz libzt/bin_macosuniversal libzt/include')
          stash includes: ('macosuniversal-' + env.BUILD_NUMBER + '.tar.gz'), name: 'mac-archive'
        }*/
        ws {
          sh('rm -Rf libzt || true')
          unstash 'win32'
          sh('mv libzt/ext/lwip/src/include/lwip libzt/include/')
          sh('mv libzt/ext/lwip-contrib/ports/win32/include/* libzt/include/lwip/')
          sh('zip -r win32-' + env.BUILD_NUMBER + '.zip libzt/bin_win32 libzt/include')
          stash includes: ('win32-' + env.BUILD_NUMBER + '.zip'), name: 'win32-archive'
        }
        ws {
          sh('rm -Rf libzt || true')
          unstash 'win64'
          sh('mv libzt/ext/lwip/src/include/lwip libzt/include/')
          sh('mv libzt/ext/lwip-contrib/ports/win32/include/* libzt/include/lwip/')
          sh('zip -r win64-' + env.BUILD_NUMBER + '.zip libzt/bin_win64 libzt/include')
          stash includes: ('win64-' + env.BUILD_NUMBER + '.zip'), name: 'win64-archive'
        }
        unstash 'linux-archive'
        //unstash 'mac-archive'
        unstash 'win32-archive'
        unstash 'win64-archive'
        sh('\$GITHUB_RELEASE upload --user HiveMP --repo zerotier-build --tag 0.' + env.BUILD_NUMBER + ' -n linux64-' + env.BUILD_NUMBER + '.tar.gz -f linux64-' + env.BUILD_NUMBER + '.tar.gz -l "libzt binaries and static libraries for Linux (64-bit)"')
        //sh('\$GITHUB_RELEASE upload --user HiveMP --repo zerotier-build --tag 0.' + env.BUILD_NUMBER + ' -n macosuniversal-' + env.BUILD_NUMBER + '.tar.gz -f macosuniversal-' + env.BUILD_NUMBER + '.tar.gz -l "libzt binaries and static libraries for macOS (Universal)"')
        sh('\$GITHUB_RELEASE upload --user HiveMP --repo zerotier-build --tag 0.' + env.BUILD_NUMBER + ' -n win32-' + env.BUILD_NUMBER + '.zip -f win32-' + env.BUILD_NUMBER + '.zip -l "libzt binaries and static libraries for Windows (32-bit)"')
        sh('\$GITHUB_RELEASE upload --user HiveMP --repo zerotier-build --tag 0.' + env.BUILD_NUMBER + ' -n win64-' + env.BUILD_NUMBER + '.zip -f win64-' + env.BUILD_NUMBER + '.zip -l "libzt binaries and static libraries for Windows (64-bit)"')
        sh('\$GITHUB_RELEASE edit --user HiveMP --repo zerotier-build --tag 0.' + env.BUILD_NUMBER + ' -n "libzt binaries (build #' + env.BUILD_NUMBER + ')" -d "These are automatically built binaries and static libraries for libzt (ZeroTier). These binaries are GPL-licensed unless you have a commercial license from ZeroTier, Inc. See the README in this repository for more information."')
      }
    }
  }
}