ifeq ($(source_url),)
	git_remote := $(shell git config branch.`git name-rev --name-only HEAD`.remote)
	git_remote := $(if $(git_remote),$(git_remote),origin)
	source_url := $(shell git config --get remote.$(git_remote).url)
	source_url := $(if $(source_url),$(source_url),https://github.com/puppetlabs/ezbake)
endif

REGISTRY ?= docker.io
REPOSITORY ?= puppet/ezbake

vcs_ref := $(shell git rev-parse HEAD)
build_date := $(shell date -u +%FT%T)
image_vendor := $(if $(IMAGE_VENDOR),$(IMAGE_VENDOR),Puppet)
image_user_name := $(if $(IMAGE_USER_NAME),$(IMAGE_USER_NAME),Puppet Release Team)
image_user_email := $(if $(IMAGE_USER_EMAIL),$(IMAGE_USER_EMAIL),release@puppet.com)

hadolint_available := $(shell hadolint --help > /dev/null 2>&1; echo $$?)
hadolint_command := hadolint --ignore DL3008 --ignore DL3018 --ignore DL3028 --ignore DL3033 --ignore DL4000 --ignore DL4001
hadolint_container := hadolint/hadolint:latest
export BUNDLE_PATH = $(PWD)/.bundle/gems
export BUNDLE_BIN = $(PWD)/.bundle/bin
export GEMFILE = $(PWD)/Gemfile

version := $(shell sed -rn 's~.*defproject .*"([^"]+)"~\1~p' ../project.clj)
dockerfile := ezbake/Dockerfile

lint: $(dockerfile)
	$(info -> $@ $<)
ifeq ($(hadolint_available),0)
	@$(hadolint_command) $<
else
	@cat $< | docker run --rm -i $(hadolint_container) $(hadolint_command) -
endif

build:
	$(info -> $@ verison $(version) [$(build_date) $(source_url) $(vcs_ref)])
	@docker build \
		--pull \
		--build-arg vcs_ref=$(vcs_ref) \
		--build-arg build_date=$(build_date) \
		--build-arg version=$(version) \
		--build-arg source_url=$(source_url) \
		--build-arg image_vendor=$(image_vendor) \
		--build-arg image_user_name=$(image_user_name) \
		--build-arg image_user_email=$(image_user_email) \
		--file $(dockerfile) \
		--tag $(REPOSITORY):$(version) \
		$(if $(findstring true,$(TAG_LATEST)),--tag $(REPOSITORY):latest,) \
		$(PWD)/..

test:
	$(info -> $@ image with rspec)
	@bundle install --path $$BUNDLE_PATH --gemfile $$GEMFILE
	@PUPPET_TEST_DOCKER_IMAGE=$(REPOSITORY):$(version) \
		bundle exec --gemfile $$GEMFILE rspec --format documentation spec

push-image:
	$(info -> $@ to $(REGISTRY)/$(REPOSITORY))
	@docker tag $(REPOSITORY):$(version) $(REGISTRY)/$(REPOSITORY):$(version)
	@docker push $(REGISTRY)/$(REPOSITORY):$(version)
ifeq ($(TAG_LATEST),true)
	@docker tag $(REPOSITORY):latest $(REGISTRY)/$(REPOSITORY):latest
	@docker push $(REGISTRY)/$(REPOSITORY):latest
endif

push-readme:
ifeq ($(REGISTRY),docker.io)
	$(info -> $@ to $(REGISTRY)/$(REPOSITORY))
	@docker run -v $(PWD):/workspace \
		-e DOCKERHUB_USERNAME="$(REGISTRY_USERNAME)" \
		-e DOCKERHUB_PASSWORD="$(REGISTRY_PASSWORD)" \
		-e DOCKERHUB_REPOSITORY=$(REPOSITORY) \
		-e README_FILEPATH='/workspace/README.md' \
		peterevans/dockerhub-description:3
endif

publish: push-image push-readme
	$(info -> $@ to $(REGISTRY))

.PHONY: lint build test publish push-image push-readme
