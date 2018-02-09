node('linux') {
  checkout scm
  sh('git submodule update --init --recursive')
  sh('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE')
  sh('cmake --build build')
}
node('mac') {
  checkout scm
  sh('git submodule update --init --recursive')
  sh('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE')
  sh('cmake --build build')
}
node('windows') {
  checkout scm
  bat('git submodule update --init --recursive')
  bat('cmake -H. -Bbuild -DCMAKE_BUILD_TYPE=RELEASE')
  bat('cmake --build build')
}