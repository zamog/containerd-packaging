#!groovy

// List of packages to build. Note that this list is overridden in the packaging
// repository, where additional variants may be added for enterprise.
//
// This list is ordered by Distro (alphabetically), and release (chronologically).
// When adding a distro here, also open a pull request in the release repository.
def images = [
    [image: "docker.io/library/amazonlinux:2",          arches: ["aarch64"]],
    [image: "docker.io/library/centos:7",               arches: ["amd64", "aarch64"]],
    [image: "quay.io/centos/centos:stream8",            arches: ["amd64", "aarch64"]],          // CentOS Stream 8 (EOL: 2024-05-31)
    [image: "quay.io/centos/centos:stream9",            arches: ["amd64", "aarch64"]],          // CentOS Stream 9 (EOL: 2027)
    [image: "docker.io/library/rockylinux:8",           arches: ["amd64", "aarch64"]],          // Rocky Linux 8 (EOL: 2029-05-31)
    [image: "docker.io/library/rockylinux:9",           arches: ["amd64", "aarch64"]],          // Rocky Linux 9 (EOL: 2032-05-31)
    [image: "docker.io/library/almalinux:8",            arches: ["amd64", "aarch64"]],          // AlmaLinux 8 (EOL: 2029)
    [image: "docker.io/library/almalinux:9",            arches: ["amd64", "aarch64"]],          // AlmaLinux 9 (EOL: 2032)
    [image: "docker.io/library/debian:buster",          arches: ["amd64", "aarch64", "armhf"]], // Debian 10 (EOL: 2022-09-10, EOL LTS: 2024-06-30)
    [image: "docker.io/library/debian:bullseye",        arches: ["amd64", "aarch64", "armhf"]], // Debian 11 (EOL: 2024)
    [image: "docker.io/library/debian:bookworm",        arches: ["amd64", "aarch64", "armhf"]], // Debian 12 (stable)
    [image: "docker.io/library/fedora:38",              arches: ["amd64", "aarch64"]],          // Fedora 38 (EOL: May 14, 2024)
    [image: "docker.io/library/fedora:39",              arches: ["amd64", "aarch64"]],          // Fedora 39 (EOL: November 12, 2024)
    [image: "docker.io/library/fedora:rawhide",         arches: ["amd64", "aarch64"]],          // Rawhide is the name given to the current development version of Fedora
    [image: "docker.io/opensuse/leap:15",               arches: ["amd64"]],
    [image: "docker.io/balenalib/rpi-raspbian:buster",  arches: ["armhf"]],
    [image: "docker.io/balenalib/rpi-raspbian:bullseye",arches: ["armhf"]],
    [image: "docker.io/balenalib/rpi-raspbian:bookworm",arches: ["armhf"]],
    [image: "docker.io/library/ubuntu:focal",           arches: ["amd64", "aarch64", "armhf"]], // Ubuntu 20.04 LTS (End of support: April, 2025. EOL: April, 2030)
    [image: "docker.io/library/ubuntu:jammy",           arches: ["amd64", "aarch64", "armhf"]], // Ubuntu 22.04 LTS (End of support: April, 2027. EOL: April, 2032)
    [image: "docker.io/library/ubuntu:mantic",          arches: ["amd64", "aarch64", "armhf"]], // Ubuntu 23.10 (EOL: July, 2024)
]

def generatePackageStep(opts, arch) {
    return {
        wrappedNode(label: "ubuntu-2004 && ${arch}") {
            stage("${opts.image}-${arch}") {
                try {
                    sh 'docker version'
                    sh 'docker info'
                    sh '''
                    curl -fsSL "https://raw.githubusercontent.com/moby/moby/master/contrib/check-config.sh" | bash || true
                    '''
                    checkout scm
                    sh 'make clean'
                    sh "make CREATE_ARCHIVE=1 ${opts.image}"
                    archiveArtifacts(artifacts: 'archive/*.tar.gz', onlyIfSuccessful: true)
                } finally {
                    deleteDir()
                }
            }
        }
    }
}

def generatePackageSteps(opts) {
    return opts.arches.collectEntries {
        ["${opts.image}-${it}": generatePackageStep(opts, it)]
    }
}

def packageBuildSteps = [
    "windows": { ->
        node("windows-2022") {
            stage("windows") {
                try {
                    checkout scm
                    sh("make -f Makefile.win archive")
                } finally {
                    deleteDir()
                }
            }
        }
    }
]

packageBuildSteps << images.collectEntries { generatePackageSteps(it) }

pipeline {
    agent none
    stages {
        stage('Check file headers') {
            agent { label 'ubuntu-2004 && amd64' }
            steps{
                script{
                    checkout scm
                    sh "make validate"
                }
            }
        }
        stage('Build packages') {
            steps {
                script {
                    parallel(packageBuildSteps)
                }
            }
        }
    }
}
