RELEASE_BRANCH := html
RELEASE_REPO := https://github.com/aleksey925/knowledge-base.git


debug-build:
	rm -rf ./output \
	&& docker-compose up debug-build

build:
	rm -rf ./output \
	&& docker-compose up build

publish:
	rm -rf ./output \
	&& docker-compose up build \
	&& cd ./output \
   	&& git init \
   	&& git config --local user.name "ci user" \
    && git config --local user.email "ci@ci.com" \
   	&& git add . \
    && git commit -m 'init' \
    && git remote add origin ${RELEASE_REPO} \
    && git branch -m ${RELEASE_BRANCH} \
    && git push -u origin ${RELEASE_BRANCH} -f
