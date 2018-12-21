/*
 * This file makes two assumptions about your Jenkins build environment:
 *
 * 1.  You have nodes set up with labels of the form "docker-${ARCH}" to
 *     support the various build architectures (currently 'x86_64',
 *     's390x', 'aarch64' (ARM), and 'ppc64le').
 * 2.  If you do not want to target the 'docker.io/openwhisk' registry
 *     (and unless you're the official maintainer, you shouldn't), then
 *     you've set up an OPENWHISK_TARGET_REGISTRY variable with the target
 *     registry you'll use.
 *
 * TODO:  Set up the build architectures as a parameter that will drive
 *        a scripted loop to build stages.
 */

def build_shell='''
target_image="${OPENWHISK_TARGET_REGISTRY:-docker.io}/${OPENWHISK_TARGET_PREFIX:-openwhisk}/php"
docker build -t "${target_image}:7.3.0-cli-stretch-$(uname -m)" ./7.3/stretch/cli
docker push "${target_image}:7.3.0-cli-stretch-$(uname -m)"
'''

def manifest_shell='''
target_image="${OPENWHISK_TARGET_REGISTRY:-docker.io}/${OPENWHISK_TARGET_PREFIX:-openwhisk}/php"
rm -rf ~/.docker/manifests
for i in 7.3.0-cli-stretch; do
  docker manifest create $target_image:$i \
    $target_image:$i-x86_64 \
    $target_image:$i-s390x \
    $target_image:$i-aarch64 \
    $target_image:$i-ppc64le
  docker manifest annotate --os linux --arch amd64 \
    $target_image:$i $target_image:$i-x86_64
  docker manifest annotate --os linux --arch s390x \
    $target_image:$i $target_image:$i-s390x
  docker manifest annotate --os linux --arch arm64 \
    $target_image:$i $target_image:$i-aarch64
  docker manifest annotate --os linux --arch ppc64le \
    $target_image:$i $target_image:$i-ppc64le
  docker manifest push $target_image:$i
done
'''

pipeline {
  agent none
  stages {
    stage('Build') {
      parallel {
        stage("Build-x86_64") {
          agent {
            label "docker-x86_64"
          }
          steps {
            sh build_shell
          }
        }
        stage("Build-s390x") {
          agent {
            label "docker-s390x"
          }
          steps {
            sh build_shell
          }
        }
        stage("Build-aarch64") {
          agent {
            label "docker-aarch64"
          }
          steps {
            sh build_shell
          }
        }
        stage("Build-ppc64le") {
          agent {
            label "docker-ppc64le"
          }
          steps {
            sh build_shell
          }
        }
      }
    }
    stage("Manifest") {
      agent {
        // Could be anything capable of running 'docker manifest', but right
        // now only the x86_64 environment is set up for that.
        label "docker-x86_64"
      }
      steps {
        sh manifest_shell
      }
    }
  }
}
