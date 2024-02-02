#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
# To edit the 'workspace' base Image, visit its repository on Github
#    https://github.com/Laradock/workspace
#
# To change its version, see the available Tags on the Docker Hub:
#    https://hub.docker.com/r/laradock/workspace/tags/
#
# Note: Base Image name format {image-tag}-{php-version}
#

ARG NODE_VERSION
FROM node:${NODE_VERSION}

# Start as root
USER root

###########################################################################
# If you're in China, or you need to change sources, will be set CHANGE_SOURCE to true in .env.
###########################################################################

ARG CHANGE_SOURCE=false
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    # Change application source from ubuntu.com to tuna.tsinghua.edu.cn source
    sed -i "s@http://.*archive.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list; \
    sed -i "s@http://.*security.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list; \
  fi;

###########################################################################
# ignition non-root user:
###########################################################################

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN groupmod -g 1001 node \
  && usermod -u 1001 -g 1001 node

RUN set -xe; \
    apt-get update -yqq && \
    groupadd -g ${PGID} ignition && \
    useradd -l -u ${PUID} -g ignition -m ignition && \
    usermod -p "*" ignition -s /bin/bash && \
    apt-get install -yqq \
      apt-utils \
      libzip-dev \
      zip \
      unzip \
      git \
      curl \
      vim \
      nano \
      postgresql-client \
      default-mysql-client \
      libedit-dev \
      libssl-dev \
      libxml2-dev \
      xz-utils \
      pkg-config \
      tree

ENV NVM_DIR /home/ignition/.nvm
RUN mkdir -p $NVM_DIR && \
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash \
        && . $NVM_DIR/nvm.sh

###########################################################################
# Set Timezone
###########################################################################

ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# User Aliases
###########################################################################

USER root

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/ignition/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/ignition/aliases.sh && \
    chown ignition:ignition /home/ignition/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	  echo "" >> ~/.bashrc

USER ignition

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	  echo "" >> ~/.bashrc

###########################################################################
# Fish Shell
###########################################################################

USER root

ARG SHELL_FISH=true
RUN if [ ${SHELL_FISH} = true ]; then \
  apt update && \
  apt-get -y install fish wget && \
  chsh -s /bin/fish ignition \
;fi

USER ignition

COPY ./aliases.sh /home/ignition/aliases.sh

RUN if [ ${SHELL_FISH} = true ]; then \
  # Install fish_ssh_agent
  mkdir -p /home/ignition/.config/fish/functions && \
  wget https://gitlab.com/kyb/fish_ssh_agent/raw/master/functions/fish_ssh_agent.fish -P /home/ignition/.config/fish/functions && \
  echo "fish_ssh_agent" > /home/ignition/.config/fish/config.fish && \
  echo "fish_ssh_agent" > /home/ignition/.config/fish/config.fish && \
  curl -sfL https://git.io/fundle-install | fish && \
  fish -c "fundle plugin 'FabioAntunes/fish-nvm' && fundle plugin 'edc/bass' && fundle install" && \
  # Setup aliases
  sed -i 's/\r//' /home/ignition/aliases.sh && \
  chown ignition:ignition /home/ignition/aliases.sh && \
  echo "fundle plugin 'FabioAntunes/fish-nvm'" >> /home/ignition/.config/fish/config.fish && \
  echo "fundle plugin 'edc/bass'" >> /home/ignition/.config/fish/config.fish && \
  echo "fundle init" >> /home/ignition/.config/fish/config.fish && \
  echo "bass source ~/aliases.sh" >> /home/ignition/.config/fish/config.fish \
;fi

###########################################################################
# STRIPE_CLI
###########################################################################

USER root

ARG INSTALL_STRIPE_CLI=false

RUN if [ ${INSTALL_STRIPE_CLI} = true ]; then \
  curl -s https://packages.stripe.dev/api/security/keypair/stripe-cli-gpg/public | gpg --dearmor | tee /usr/share/keyrings/stripe.gpg && \
  echo "deb [signed-by=/usr/share/keyrings/stripe.gpg] https://packages.stripe.dev/stripe-cli-debian-local stable main" | tee -a /etc/apt/sources.list.d/stripe.list && \
  apt update && \
  apt install stripe \
;fi

###########################################################################
# VS Code Configuration
###########################################################################

ARG WORKSPACE_VSCODE_DEVCONTAINER

LABEL devcontainer.metadata="${WORKSPACE_VSCODE_DEVCONTAINER}"

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm -f /var/log/lastlog /var/log/faillog

# Set default work directory
WORKDIR /var/www
