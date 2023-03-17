FROM ubuntu:latest
ENV REFRESHED_AT 17-03-2023

RUN apt update -y

# SOME BASIC TOOLS
RUN apt install -y ca-certificates curl gnupg lsb-release
RUN mkdir -m 0755 -p /etc/apt/keyrings

# ADD DOCKER'S OFFICIAL GPG KEY
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# SET UP THE REPOSITORY
RUN echo deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
     https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable > \ 
        /etc/apt/sources.list.d/docker.list
RUN apt update -y

# INSTALL docker 
RUN apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

ENTRYPOINT ["/bin/bash"]
