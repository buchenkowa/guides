###############################################################################
#                                                                             #
#                                                                             #
# PUBLIC CONFIG                                                               #
#                                                                             #
#                                                                             #
###############################################################################

APRESS_GEMS_CREDENTIALS  = not_set

APRESS_DUMPS_CREDENTIALS = not_set

WORKDIR                 := `pwd`

RSA_KEY                  = `echo ~`/.ssh/id_rsa


###############################################################################
#                                                                             #
#                                                                             #
# PRIVATE CONFIG                                                              #
#                                                                             #
#                                                                             #
###############################################################################

VERSION                  = 1.2.0

BLOCK_SEPARATOR          = ==============================================================================

STEP_MESSAGE             = "\n$(BLOCK_SEPARATOR)\n %s.\n$(BLOCK_SEPARATOR)\n\n"

EXPORT_DUMPS_CREDENTIALS = $(if $(shell [ ! $(APRESS_DUMPS_CREDENTIALS) = not_set ] && echo true),\
                                APRESS_DUMPS_CREDENTIALS=$(APRESS_DUMPS_CREDENTIALS))

IS_REBOOT_NEEDED         = false


###############################################################################
#                                                                             #
#                                                                             #
# HELP DOC                                                                    #
#                                                                             #
#                                                                             #
###############################################################################

help:
	@echo "\
		\r\n\033[1m DESCRIPTION: \033[0m \n\
		\
		\n\r     this makefile provides deploy one of the apress project in the Docker container \n\
		\
		\r\n\033[1m USAGE: \033[0m \n\
		\
		\n\r     make PROJECT_NAME [VARIABLE=VALUE]... \n\
		\
		\n\r     project names: \n\
		\n\r         blizko          <https://github.com/abak-press/blizko> \n\
		\n\r         pulscen         <https://github.com/abak-press/pulscen> \n\
		\n\r         lookmart        <https://github.com/abak-press/lookmart>\n\
		\
		\n\r     available variables:\n\
		\n\r         APRESS_GEMS_CREDENTIALS=username:password    set keys for <http://gems.railsc.ru> \
		\n\r                                                      (default - not set) \n\
		\
		\n\r         APRESS_DUMPS_CREDENTIALS=username:password   set keys for <http://dumps.railsc.ru> \
		\n\r                                                      (default - not set) \n\
		\
		\n\r         WORKDIR=dir_path                             set parent dir for project \
		\n\r                                                      (default - current dir) \n\
		\
		\n\r         RSA_KEY=rsa_key_path                         set full path to private rsa key\
		\n\r                                                      (default - ~/.ssh/id_rsa) \n\
		\
		\r\n\033[1m VERSION: \033[0m \n\
		\
		\n\r     $(VERSION) \n\
		\
		\n\r     See changelog <https://github.com/abak-press/guides/blob/master/docker-howto/CHANGELOG.md> \n\
		\
		\r\n\033[1m REPORTING BUGS: \033[0m \n\
		\
		\n\r     Report bugs to <https://github.com/abak-press/guides/issues> \n\
		\
		\r\n\033[1m AUTHOR: \033[0m \n\
		\
		\n\r     Written by Ivan Bolshakov <idbolshakov@gmail.com> \n\
		\
		\r\n\033[1m SEE ALSO: \033[0m \n\
		\
		\n\r     Full documentation at: <https://github.com/abak-press/guides/tree/master/docker-howto>\n";


###############################################################################
#                                                                             #
#                                                                             #
# PUBLIC TARGETS                                                              #
#                                                                             #
#                                                                             #
###############################################################################


##
# clone_project function
#
# $1 - project name
# $2 - project docker readme link
##
define deploy_complete_message

	@printf "\n ${1} deployment complete.\n If you don't know what you should do now, please read:\n <${2}>\n\n";

	@if [ $(IS_REBOOT_NEEDED) = true ]; then\
	  printf " System need to reboot. Reboot right now (y/n)? ";\
	  read answer;\
	  if [ $$answer = y ]; then\
	    sudo reboot;\
	  else\
	    printf " Please reboot your system before using project\n\n";\
	  fi;\
	fi;
endef

blizko: docker docker-compose dip blizko-clone blizko-dumps ssh dns blizko-provision
	$(call deploy_complete_message,blizko,https://github.com/abak-press/blizko/blob/develop/docker/README.md#webric)

pulscen: docker docker-compose dip pulscen-clone pulscen-dumps ssh dns pulscen-provision
	$(call deploy_complete_message,pulscen,https://github.com/abak-press/pulscen/blob/develop/docker/README.md#webric)

lookmart: docker docker-compose dip lookmart-clone lookmart-dumps ssh dns lookmart-provision
	$(call deploy_complete_message,lookmart,https://github.com/abak-press/lookmart/blob/develop/docker/README.md#webric)

###############################################################################
#                                                                             #
#                                                                             #
# PRIVATE TARGETS                                                             #
#                                                                             #
#                                                                             #
###############################################################################

##
# docker install
#
# https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-docker-ce
##
docker:
	@printf $(STEP_MESSAGE) 'BEGIN install docker CE';

	  @sudo apt-get update;\
	  sudo apt-get install apt-transport-https ca-certificates curl software-properties-common;\
	  curl -fsSL https://download.docker.com/linux/ubuntu/gpg 2>&1 |  sudo apt-key add -;\
	  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $$(lsb_release -cs) stable";\
	  sudo apt-get update;\
	  sudo apt-get install docker-ce;\
	  sudo groupadd docker;\
	  sudo usermod -aG docker `echo $$USER`;

	  $(eval IS_REBOOT_NEEDED = true)

	@printf $(STEP_MESSAGE) 'END install docker CE';



##
# docker-compose install
##
docker-compose:
	@printf $(STEP_MESSAGE) 'BEGIN install docker-compose';

	  @sudo apt-get install docker-compose;

	@printf $(STEP_MESSAGE) 'END install docker-compose';



##
# dip install
#
# https://github.com/bibendi/dip
##
dip:
	@printf $(STEP_MESSAGE) 'BEGIN install dip';

	  @if [ ! $(shell command -v dip 2>/dev/null) ]; then\
	    curl -L https://github.com/bibendi/dip/releases/download/1.0.2/dip-`uname -s`-`uname -m` > "$(WORKDIR)/dip";\
	    sudo mv "$(WORKDIR)/dip" /usr/local/bin/dip;\
	    sudo chmod +x /usr/local/bin/dip;\
	  else\
	    echo 'dip is already installed';\
	  fi;

	@printf $(STEP_MESSAGE) 'END install dip';


##
# clone_project function
#
# $1 - project name
##
define clone_project
	@printf $(STEP_MESSAGE) 'BEGIN clone $(1)';

	@if [ "$(shell ls -A "$(WORKDIR)/$(1)" 2>/dev/null)" ]; then\
	  echo project directory already exists.;\
	else\
	  git clone "https://github.com/abak-press/$(1)" "$(WORKDIR)/$(1)";\
	  cd "$(WORKDIR)/$(1)";\
	  git remote rename origin upstream;\
	fi;

	@printf $(STEP_MESSAGE) 'END clone $(1)';
endef

blizko-clone: git
	$(call clone_project,blizko)

pulscen-clone: git
	$(call clone_project,pulscen)

lookmart-clone: git
	$(call clone_project,lookmart)

git: 
	@printf $(STEP_MESSAGE) 'BEGIN install git';

	  @sudo apt-get install git;

	@printf $(STEP_MESSAGE) 'END install git';

##
# download_dumps function
#
# $1 - project name
# $2 - download dumps script name
##
define download_dumps
	@sg docker -c 'cd "$(WORKDIR)/$(1)"; $(EXPORT_DUMPS_CREDENTIALS) ./docker/bin/$(2)'
endef

##
# download blizko dumps
##
blizko-dumps:
	@printf $(STEP_MESSAGE) 'BEGIN download blizko dumps';

	  $(call download_dumps,blizko,restore)

	@printf $(STEP_MESSAGE) 'END download blizko dumps';



##
# download pulscen dumps
##
pulscen-dumps:
	@printf $(STEP_MESSAGE) 'BEGIN download pulscen dumps';

	  $(call download_dumps,pulscen,restore_dumps)

	@printf $(STEP_MESSAGE) 'END download pulscen dumps';



##
# download lookmart dumps
##
lookmart-dumps:
	@printf $(STEP_MESSAGE) 'BEGIN download lookmart dumps';

	  $(call download_dumps,lookmart,restore_dumps)

	@printf $(STEP_MESSAGE) 'END download lookmart dumps';



##
# start ssh agent
##
ssh:
	@printf $(STEP_MESSAGE) 'BEGIN init ssh agent';

	  @printf\
	    "ssh agent expecting generated public/private rsa key pair ($(RSA_KEY) & $(RSA_KEY).pub).\n\
	    \rPublic part of this rsa key should be added to your github account. \n\
	    \rIf you don't know how did this, please read: \n\
	    \r<https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/>\n\n\
	    \rGenerate new public/private rsa key pair ($(RSA_KEY) & $(RSA_KEY).pub) (y/n)? ";\
	  read answer;\
	  if [ $$answer = y ]; then\
	    ssh-keygen -f "$(RSA_KEY)" -t rsa;\
	  else\
	    printf "\nnew public/private rsa key pair doesn't generated\n";\
	  fi;\
	  printf\
	    "\nPlease make sure, that public part of rsa ($(RSA_KEY).pub) \n\
	    \radded to your github account and then press ENTER...";\
	  read answer; printf "\n";\
	  sg docker -c 'dip ssh down' 2>&1 | tee /dev/null;
	  sg docker -c 'dip ssh add -k "$(RSA_KEY)"' 2>&1 | tee /dev/null;

	@printf $(STEP_MESSAGE) 'END init ssh agent';



##
# start dns server
##
dns:
	@printf $(STEP_MESSAGE) 'BEGIN init dns server';

	  @sg docker -c 'dip dns up' 2>&1 | tee /dev/null;\
	  dnsdock_port=`sg docker -c 'docker port dnsdock'`;\
	  without_prefix=`echo $${dnsdock_port#'53/udp -> '}`;\
	  dnsdock_ip=`echo $${without_prefix%':53'}`;\
	  echo server=/docker/$$dnsdock_ip | sudo tee -a /etc/NetworkManager/dnsmasq.d/dnsmasq.conf;\
	  sudo service network-manager restart;

	@printf $(STEP_MESSAGE) 'END init dns server';



##
# provision function
#
# $1 - project name
##
define provision
	@printf $(STEP_MESSAGE) 'BEGIN $(1) provision';

	  @cd "$(WORKDIR)/$(1)";\
	  sg docker -c 'dip compose down';\
	  if [ ! $(APRESS_GEMS_CREDENTIALS) = not_set ]; then\
	    export APRESS_GEMS_CREDENTIALS=$(APRESS_GEMS_CREDENTIALS);\
	  fi;\
	  sg docker -c 'dip provision';

	@printf $(STEP_MESSAGE) 'END $(1) provision';
endef

blizko-provision:
	$(call provision,blizko)

pulscen-provision:
	$(call provision,pulscen)

lookmart-provision:
	$(call provision,lookmart)
