

def avocado_test_url = 'https://github.com/dmonakhov/avocado-misc-tests.git' ;
def avocado_url = 'https://github.com/dmonakhov/avocado.git';
def avocado_branch = 'inst';
def REGISTRY = 'alice.qa.sw.ru:5000';


stage 'Checkout sources'

node () {
    deleteDir();
    checkout scm;
    dir('avocado-misc-tests') {
	git url: avocado_test_url, branch: avocado_branch;
    }
    dir('avocado-framework') {
	git url: avocado_url, branch: avocado_branch;
	sh 'which inspekt || pip install inspektor';
    }
    stash name: 'sources', includes: 'avocado-misc-tests/,avocado-framework/,Makefile.avocado'
}
stage 'Check syntax';
job=[:]


build_jobs["Static check: avocado-misc-tests"] = {
    node {
	dir('bld') {
	    deleteDir();
	    unstash "sources"
	    dir('avocado-misc-tests') {
		sh 'which inspekt || pip install inspektor'
		sh 'make -j `nproc` check -f ../Makefile.avocado'
	    }
	}
    }
}
build_jobs["Static check: avocado-framework'"] = {
    node {
	dir('bld') {
	    deleteDir();
	    unstash "sources"
	    dir('avocado-framework') {
		sh 'which inspekt || pip install inspektor'
		//// TODO FIX sources
		sh 'make -j `nproc` check -f ../Makefile.avocado || /bin/true'
	    }
	}
    }
}
build_jobs["Build docker image"] = {
    node () {
	dir('bld') {
	    deleteDir();
	    unstash "sources"
	    dir('avocado-framework') {
		sh 'docker build --force-rm -t debian-avocado -f contrib/docker/Dockerfile.debian .'
		sh 'docker tag debian-avocado ${REGISTRY}/debian-avocado'
		// TODO Require uptodate inventory file
		//ansible avocado -a 'docker pull $REGISTRY/debian-avocado'

		//# This is cleanup stage it should not affect whole deployment
		sh 'docker rmi debian-avocado'
		//ansible avocado -m shell -a 'docker  rmi $(docker images -f "dangling=true" $REGISTRY/debian-avocado -q)' || /bin/true
	    }
	}
    }
    //	stage 'build and deploy docker image'
    //	sh 'avocado-deploy/deploy-docker-image.sh'
    //	stage 'Deploy host instalation'
    //	sh 'ansible-playbook avocado-deploy/ansible-deploy-avocado.yaml'
}

stage 'build and deploy docker image'
