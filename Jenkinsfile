

def avocado_test_url = 'https://github.com/dmonakhov/avocado-misc-tests.git' ;
def avocado_url = 'https://github.com/dmonakhov/avocado-misc-tests.git';
def avocado_branch = 'inst';
def REGISTRY = 'alice.qa.sw.ru:5000';


node {
	stage 'Fetch source and run tests'
	dir('avocado-misc-tests') {
		deleteDir()
		git url: avocado_test_url, branch: avocado_branch;
		sh 'which inspekt || pip install inspektor'
		sh 'make -j `nproc` check -f ../Makefile.avocado'
	}
	dir('avocado-framework') {
		deleteDir()
		git url: avocado_test_url, branch: avocado_branch;
		sh 'which inspekt || pip install inspektor';
		sh 'make -j `nproc` check -f ../Makefile.avocado'
	}
	stash name: 'sources', includes: 'avocado-misc-tests/,avocado-framework/'
	stage 'build and deploy docker image'
	dir('avocado-framework') {
		sh 'docker build --force-rm -t debian-avocado -f contrib/docker/Dockerfile.debian'
		sh 'docker tag debian-avocado ${REGISTRY}/debian-avocado'
		// TODO Require uptodate inventory file
		//ansible avocado -a 'docker pull $REGISTRY/debian-avocado'

		//# This is cleanup stage it should not affect whole deployment
		sh 'docker rmi debian-avocado'
		//ansible avocado -m shell -a 'docker  rmi $(docker images -f "dangling=true" $REGISTRY/debian-avocado -q)' || /bin/true
	}
//	stage 'build and deploy docker image'
//	sh 'avocado-deploy/deploy-docker-image.sh'
//	stage 'Deploy host instalation'
//	sh 'ansible-playbook avocado-deploy/ansible-deploy-avocado.yaml'
}

