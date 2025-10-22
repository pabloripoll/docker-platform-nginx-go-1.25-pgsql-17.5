# This Makefile requires GNU Make.
MAKEFLAGS += --silent

# Settings
ifeq ($(strip $(OS)),Windows_NT) # is Windows_NT on XP, 2000, 7, Vista, 10...
    DETECTED_OS := Windows
	C_BLU=''
	C_GRN=''
	C_RED=''
	C_YEL=''
	C_END=''
else
    DETECTED_OS := $(shell uname) # same as "uname -s"
	C_BLU='\033[0;34m'
	C_GRN='\033[0;32m'
	C_RED='\033[0;31m'
	C_YEL='\033[0;33m'
	C_END='\033[0m'
endif

include .env

APIREST_BRANCH:=develop
APIREST_PROJECT:=$(PROJECT_NAME) - APIREST
APIREST_CONTAINER:=$(addsuffix -$(APIREST_CAAS), $(PROJECT_LEAD))

DATABASE_CONTAINER:=$(addsuffix -$(DATABASE_CAAS), $(PROJECT_LEAD))

ROOT_DIR=$(patsubst %/,%,$(dir $(realpath $(firstword $(MAKEFILE_LIST)))))
DIR_BASENAME=$(shell basename $(ROOT_DIR))

.PHONY: help

# -------------------------------------------------------------------------------------------------
#  Help
# -------------------------------------------------------------------------------------------------

help: ## shows this Makefile help message
	echo "Usage: $$ make "${C_GRN}"[target]"${C_END}
	echo ${C_GRN}"Targets:"${C_END}
	awk 'BEGIN {FS = ":.*?## "} /^[a-zA-Z0-9_-]+:.*?## / {printf "$$ make \033[0;33m%-30s\033[0m %s\n", $$1, $$2}' ${MAKEFILE_LIST} | column -t -c 2 -s ':#'

# -------------------------------------------------------------------------------------------------
#  System
# -------------------------------------------------------------------------------------------------
.PHONY: local-hostname local-ownership local-ownership-set

local-hostname: ## shows local machine ip and container ports set
	echo "Container Address:"
	echo ${C_BLU}"LOCAL: "${C_END}"$(word 1,$(shell hostname -I))"
	echo ${C_BLU}"APIREST: "${C_END}"$(word 1,$(shell hostname -I)):"$(APIREST_PORT)
	echo ${C_BLU}"DATABASE: "${C_END}"$(word 1,$(shell hostname -I)):"$(DATABASE_PORT)

user ?= ${USER}
group ?= root
local-ownership: ## shows local ownership
	echo $(user):$(group)

local-ownership-set: ## sets recursively local root directory ownership
	$(SUDO) chown -R ${user}:${group} $(ROOT_DIR)/

# -------------------------------------------------------------------------------------------------
#  Backend API Rest Service
# -------------------------------------------------------------------------------------------------
.PHONY: apirest-hostcheck apirest-info apirest-set apirest-create apirest-network apirest-ssh apirest-start apirest-stop apirest-destroy

apirest-hostcheck: ## shows this project ports availability on local machine for apirest container
	cd platform/$(APIREST_PLTF) && $(MAKE) port-check

apirest-info: ## shows the apirest docker related information
	cd platform/$(APIREST_PLTF) && $(MAKE) info

apirest-set: ## sets the apirest enviroment file to build the container
	cd platform/$(APIREST_PLTF) && $(MAKE) env-set

apirest-create: ## creates the apirest container from Docker image
	cd platform/$(APIREST_PLTF) && $(MAKE) build up

apirest-network: ## creates the apirest container network - execute this recipe first before others
	$(MAKE) apirest-stop
	cd platform/$(APIREST_PLTF) && $(DOCKER_COMPOSE) -f docker-compose.yml -f docker-compose.network.yml up -d

apirest-ssh: ## enters the apirest container shell
	cd platform/$(APIREST_PLTF) && $(MAKE) ssh

apirest-start: ## starts the apirest container running
	cd platform/$(APIREST_PLTF) && $(MAKE) start

apirest-stop: ## stops the apirest container but its assets will not be destroyed
	cd platform/$(APIREST_PLTF) && $(MAKE) stop

apirest-destroy: ## destroys completly the apirest container
	echo ${C_RED}"Attention!"${C_END};
	echo ${C_YEL}"You're about to remove the "${C_BLU}"$(APIREST_PROJECT)"${C_END}" container and delete its image resource."${C_END};
	@echo -n ${C_RED}"Are you sure to proceed? "${C_END}"[Y/n]: " && read response && if [ $${response:-'n'} != 'Y' ]; then \
        echo ${C_GRN}"K.O.! container has been stopped but not destroyed."${C_END}; \
    else \
		cd platform/$(APIREST_PLTF) && $(MAKE) stop clear destroy; \
		echo -n ${C_GRN}"Do you want to clear DOCKER cache? "${C_END}"[Y/n]: " && read response && if [ $${response:-'n'} != 'Y' ]; then \
			echo ${C_YEL}"The following command is delegated to be executed by user:"${C_END}; \
			echo "$$ $(DOCKER) system prune"; \
		else \
			$(DOCKER) system prune; \
			echo ${C_GRN}"O.K.! DOCKER cache has been cleared up."${C_END}; \
		fi \
	fi

.PHONY: apirest-git-ssh apirest-git-ssh-custom

gitkey := $(notdir $(APIREST_GIT_SSH))
apirest-git-ssh: ## copies environment default ssh-key from local machine into the container user home/.ssh directory
	if [ -z "${APIREST_GIT_SSH}" ]; then \
		echo ${C_YEL}"Set the repository SSH key file path/name on ./.env file as for .e.g: APIREST_GIT_SSH=\"~/.ssh/id_ed25519\""${C_END}; \
	else \
		if [ ! -f $(APIREST_GIT_SSH) -o ! -f $(APIREST_GIT_SSH).pub ]; then \
			echo ${C_RED}"File $(APIREST_GIT_SSH) cannot be reached or one of them is missing!"${C_END}; \
		else \
			echo "Local SSH key: "${C_YEL}"$(gitkey)"${C_END}" and "${C_YEL}"$(gitkey).pub"${C_END}; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c '[ -d ~/.ssh ] || mkdir ~/.ssh; chmod 700 ~/.ssh'; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c '[ ! -f ~/.ssh/$(gitkey) ] || rm -f ~/.ssh/$(gitkey)'; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c "cat > ~/.ssh/$(gitkey)" < $(APIREST_GIT_SSH); \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c "chmod 400 ~/.ssh/$(gitkey)"; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c '[ ! -f ~/.ssh/$(gitkey).pub ] || rm -f ~/.ssh/$(gitkey).pub'; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c "cat > ~/.ssh/$(gitkey).pub" < $(APIREST_GIT_SSH).pub; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c "chmod 400 ~/.ssh/$(gitkey).pub"; \
			echo "Creating "${C_YEL}"~/.ssh/$(gitkey)"${C_END}" and "${C_YEL}"~/.ssh/$(gitkey).pub"${C_END}; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c "ssh-keyscan $(APIREST_GIT_HOST) >> ~/.ssh/known_hosts"; \
			echo "SSH key "${C_GRN}"$(gitkey)"${C_END}" and "${C_GRN}"$(gitkey).pub"${C_END}" have been transferred from local machine to "${C_GRN}"$(APIREST_CONTAINER)"${C_END}" container."; \
		fi \
	fi

file := ""
filename := $(notdir $(file))
pathfile := $(shell dirname $(basename $(file)))
apirest-git-ssh-custom: ## copies specific ssh-key from local machine into the container user home/.ssh directory. for e.g. file=PATH/id_ed25519 (not .pub!)
	if [ $${file:-''} = "" ]; then \
		echo ${C_YEL}"ssh-key file name ir required as parameter, for .e.g: file=\"PATH/id_ed25519\""${C_END}; \
		echo "Thus, you should execute this recipe as $$ "${C_GRN}'make apirest-ssh-key file="PATH/FILE"'${C_END}; \
    else \
		if [ ! -f $(file) -o ! -f $(file).pub ]; then \
			echo ${C_RED}"File $(file) cannot be reached or one of them is missing!"${C_END}; \
		else \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c '[ -d ~/.ssh ] || mkdir ~/.ssh; chmod 700 ~/.ssh'; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c '[ ! -f ~/.ssh/$(filename) ] || rm -f ~/.ssh/$(filename)'; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c 'cat > ~/.ssh/$(filename)' < $(file); \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c 'chmod 400 ~/.ssh/$(filename)'; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c '[ ! -f ~/.ssh/$(filename).pub ] || rm -f ~/.ssh/$(filename).pub'; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c 'cat > ~/.ssh/$(filename).pub' < $(file).pub; \
			$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c 'chmod 400 ~/.ssh/$(filename).pub'; \
			echo "SSH keys transfered from local machine to "${C_GRN}"$(APIREST_CONTAINER)"${C_END}" container............"${C_GRN}"[OK]"${C_END}; \
			echo "As local "${C_YEL}$(file)${C_END}" key has already access granted to the Git-based host provider, the container "${C_GRN}~/.ssh/$(filename)${C_END}" key will not need to be registered on it."; \
			echo -n ${C_YEL}"Though is not required, would you like to test container ssh connection now?"${C_END}" [Y/n]: " && read response && if [ $${response:-'n'} != 'Y' ]; then \
				echo "Next time, when you would access into the container you can execute the following commands to test ssh connection:"; \
				echo ${C_BLU}"$$ ssh-agent /bin/sh"${C_END}; \
				echo ${C_BLU}"$$ ssh-add ~/.ssh/$(filename)"${C_END}; \
				echo ${C_BLU}"$$ ssh-keyscan -H bitbucket.com"${C_END}; \
				echo ${C_BLU}"$$ ssh -Tvv git@bitbucket.org"${C_END}; \
				echo ${C_GRN}"Process finished!"${C_END}; \
			else \
				echo ${C_GRN}"Entering into $(APIREST_CONTAINER) container tty.....................................[OK]"${C_END}; \
				echo "Executing on background:"; \
				echo ${C_BLU}"$$ ssh-agent /bin/sh"${C_END}; \
				echo ${C_BLU}"$$ ssh-add ~/.ssh/$(filename)"${C_END}; \
				echo ${C_BLU}"$$ ssh-keyscan -H bitbucket.com"${C_END}; \
				echo ${C_BLU}"$$ ssh -Tvv git@bitbucket.org"${C_END}; \
				echo "Checking if provider grants access permission to ~/.ssh/$(filename) key ssh-agent TTY..."${C_YEL}"[I/O]"${C_END}; \
				echo ${C_YEL}"Close the following TTY typing "${C_END}${C_GRN}"exit"${C_END}${C_YEL}" command on each of the following two connections to continue:"${C_END}; \
				${MAKE} apirest-ssh && ssh-agent /bin/sh && ssh-add ~/.ssh/$(filename) && ssh-keyscan -H bitbucket.com && ssh -Tvv git@bitbucket.org; \
				echo "................................................................................"${C_BLU}"[connection-closed]"${C_END}; \
				echo "If above latest line return is "${C_YEL}"\"debug1: Exit status 0\""${C_END}", then the repository access is granted."; \
				echo ${C_GRN}"Process finished!"${C_END}; \
			fi \
		fi \
	fi

.PHONY: apirest-install apirest-ownership apirest-reset-cache apirest-clear-logs

apirest-install: ## installs the latest application dependency packages
	echo ${C_YEL}"$$ composer install"${C_END}" is going to be execute inside "${C_BLU}"$(APIREST_CONTAINER)"${C_END}" running container.";
	@echo -n "Are you sure to proceed? [Y/n]: " && read response && if [ $${response:-'n'} != 'Y' ]; then \
        echo ${C_GRN}"K.O.! action has been stopped."${C_END}; \
    else \
		cd platform/$(APIREST_PLTF) && $(MAKE) app-install; \
		echo ${C_GRN}"O.K! action has been executed."${C_END}; \
	fi

apirest_user := ${APIREST_CAAS_USER}
apirest_group := ${APIREST_CAAS_GROUP}
apirest-ownership: ## sets recursively containerp project root directory ownership
	echo $(apirest_user):$(apirest_group)

apirest-reset-cache: ## sets project directory permission and ownership inside container
	echo ${C_BLU}"$(APIREST_PROJECT)"${C_END}" "${C_YEL}"maintenance:"${C_END};
	$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c 'sudo chown -R ${apirest_user}:${apirest_group} * .*;'
	$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c 'sudo chown -R ${apirest_user}:${apirest_user} bootstrap/cache';
	$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c 'sudo chmod -R ug+rwx storage bootstrap/cache';
	echo "Directories and Files "${C_YEL}"CHMOD and CHOWN reset"${C_END}"..."${C_GRN}"[OK]"${C_END};
	$(DOCKER) exec -it $(APIREST_CONTAINER) sh -c 'php artisan config:cache; php artisan config:clear';
	echo "Laravel's preset cache clearing commands......"${C_GRN}"[OK]"${C_END};

apirest-clear-logs: ## clears up laravel.log
	echo ${C_BLU}"$(APIREST_PROJECT)"${C_END}" "${C_YEL}"maintenance:"${C_END};
	$(DOCKER) exec -i $(APIREST_CONTAINER) sh -c 'echo -n "" > storage/logs/laravel.log';
	echo "Laravel's storage/logs/laravel.log cleared...."${C_GRN}"[OK]"${C_END};

# -------------------------------------------------------------------------------------------------
#  Repository Helper
# -------------------------------------------------------------------------------------------------
.PHONY: repo-flush repo-commit

repo-flush: ## clears local git repository cache specially for updating .gitignore on local IDE
	git rm -rf --cached .; git add .; git commit -m "fix: cache cleared for untracked files"

repo-commit: ## echoes common git commands
	echo "git add . && git commit -m \"feat: ... \" && git push -u origin [branch]"
	echo ${C_YEL}"For fixing pushed commit comment:"${C_END}
	echo "git commit --amend"
	echo "git push --force origin [branch]"
