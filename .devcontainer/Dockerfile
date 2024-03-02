FROM mcr.microsoft.com/devcontainers/base:alpine-3.19

RUN apk --update add jekyll ruby-dev && \
    echo "export GEM_HOME=/home/vscode/.gem" >> /home/vscode/.zshrc && \
    echo "export GEM_HOME=/home/vscode/.gem" >> /home/vscode/.bashrc
