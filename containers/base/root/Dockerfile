ARG BASE_OPENSUSE_VERSION=15.5
FROM docker.io/opensuse/leap:${BASE_OPENSUSE_VERSION} AS base

ARG BASE_OPENSUSE_VERSION
ARG BASE_GCC_VERSION
ARG BASE_BASH_VERSION
ARG BASE_GIT_VERSION
ARG BASE_DOCKER_VERSION
ARG BASE_COMPOSE_VERSION

RUN zypper --non-interactive refresh

FROM base AS core-dependencies

# Bring ARGs into this stage to use them.
ARG BASE_OPENSUSE_VERSION
ARG BASE_GCC_VERSION
ARG BASE_BASH_VERSION
ARG BASE_GIT_VERSION
ARG BASE_DOCKER_VERSION
ARG BASE_COMPOSE_VERSION

# These are the core packages for the environment. They are installed in their own stage
# as they are expected to change infrequently.
RUN zypper --non-interactive addrepo "https://download.opensuse.org/repositories/Virtualization:containers/${BASE_OPENSUSE_VERSION}/Virtualization:containers.repo" && \
    zypper --non-interactive --gpg-auto-import-keys refresh && \
    zypper --non-interactive install \
        git-${BASE_GIT_VERSION} \
        gcc-${BASE_GCC_VERSION} \
        bash-${BASE_BASH_VERSION} \
        gzip emacs libgpm2 tar wget && \
    zypper clean --all

ENV PATH="/root/.local/bin:$PATH"

# Add the non-essential user-friendly tools and configurations for students.
# Separated from core because these components are more likely to change.
FROM core-dependencies AS student-environment

# Set up fzf. Install alternative shells, prompt generator, blesh requirements, etc
RUN wget https://github.com/junegunn/fzf/releases/download/v0.62.0/fzf-0.62.0-linux_amd64.tar.gz && \
    tar xzf fzf-0.62.0-linux_amd64.tar.gz && \
    rm fzf-0.62.0-linux_amd64.tar.gz && \
    mv fzf /usr/bin/fzf && \
    zypper --non-interactive install zsh fish hashdeep jq starship gawk make wget glow lsd fd \
        fd-bash-completion ripgrep ripgrep-bash-completion tealdeer tealdeer-bash-completion \
        symbols-only-nerd-fonts vim docker-bash-completion python312-pipx && \
    zypper clean --all && \
    pipx install dotenv-cli

# Set up blesh for bash shell
WORKDIR /root
RUN git clone https://github.com/akinomyoga/ble.sh.git blesh
WORKDIR /root/blesh
RUN make install
WORKDIR /root/

# Clean up blesh folder and set up zsh shell
RUN rm -r blesh && \
    bash -c "$(wget https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh -O -)" "" --unattended && \
    sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting && \
    git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions

# Pre-install vim plugins
COPY .vimrc .vimrc
RUN vim +'PlugInstall --sync' +qa

# Move across test files and make them executable
COPY ./test/ /test/
COPY ./.config/micro/test.sh /test/base/editor_tests/micro_test.sh
COPY ./.config/emacs/test.sh /test/base/editor_tests/micro_test.sh
COPY ./.config/nvim/test.sh /test/base/editor_tests/micro_test.sh
RUN chmod -R +x /test/

COPY .bashrc .bashrc
COPY .blerc .blerc
COPY .zshrc .zshrc
COPY .config/starship .config/starship
COPY .config/glow .config/glow
COPY .config/tealdeer .config/tealdeer
COPY .config/fish .config/fish
COPY .config/nushell .config/nushell

COPY --from=ghcr.io/nushell/nushell:0.105.1-bookworm /bin/nu /bin/nu_plugin* /usr/local/bin/

COPY --from=ghcr.io/amileo/csc1109-base-root-config-bat:latest /usr/bin/bat /usr/bin/bat
COPY --from=ghcr.io/amileo/csc1109-base-root-config-bat:latest /usr/share/man/man1/bat.1.gz /usr/share/man/man1/bat.1.gz
COPY --from=ghcr.io/amileo/csc1109-base-root-config-bat:latest /usr/local/ /usr/local/
COPY --from=ghcr.io/amileo/csc1109-base-root-config-micro:latest /usr/bin/micro /usr/bin/micro
COPY --from=ghcr.io/amileo/csc1109-base-root-config-micro:latest /usr/local/ /usr/local/
COPY --from=ghcr.io/amileo/csc1109-base-root-config-emacs:latest /usr/bin/emacs /usr/bin/emacs
COPY --from=ghcr.io/amileo/csc1109-base-root-config-emacs:latest /usr/bin/emacsclient /usr/bin/emacsclient
COPY --from=ghcr.io/amileo/csc1109-base-root-config-emacs:latest /usr/bin/emacs-nox /usr/bin/emacs-nox
COPY --from=ghcr.io/amileo/csc1109-base-root-config-emacs:latest /usr/lib/emacs /usr/lib/emacs
COPY --from=ghcr.io/amileo/csc1109-base-root-config-emacs:latest /usr/share/emacs /usr/share/emacs
COPY --from=ghcr.io/amileo/csc1109-base-root-config-emacs:latest /var/games/emacs /var/games/emacs
COPY --from=ghcr.io/amileo/csc1109-base-root-config-emacs:latest /usr/share/man/man1/emacs.1.gz /usr/share/man/man1/emacs.1.gz
COPY --from=ghcr.io/amileo/csc1109-base-root-config-emacs:latest /usr/share/info/emacs.info.gz /usr/share/info/emacs.info.gz
COPY --from=ghcr.io/amileo/csc1109-base-root-config-emacs:latest /usr/local/ /usr/local/
COPY --from=ghcr.io/amileo/csc1109-base-root-config-nvim:latest /bin/nvim /bin/nvim
COPY --from=ghcr.io/amileo/csc1109-base-root-config-nvim:latest /lib/nvim /lib/nvim
COPY --from=ghcr.io/amileo/csc1109-base-root-config-nvim:latest /share/nvim /share/nvim
COPY --from=ghcr.io/amileo/csc1109-base-root-config-nvim:latest /share/man /share/man
COPY --from=ghcr.io/amileo/csc1109-base-root-config-nvim:latest /usr/local/ /usr/local/

COPY --from=ghcr.io/amileo/csc1109-base-root-config-glamour:latest /root/ /root/
COPY --from=ghcr.io/amileo/csc1109-base-root-config-bat:latest /root/ /root/
COPY --from=ghcr.io/amileo/csc1109-base-root-config-micro:latest /root/ /root/
COPY --from=ghcr.io/amileo/csc1109-base-root-config-emacs:latest /root/ /root/
COPY --from=ghcr.io/amileo/csc1109-base-root-config-nvim:latest /root/ /root/

COPY ./CSC1109.md /root/CSC1109.md
