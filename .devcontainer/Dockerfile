FROM python:3

ARG USERNAME=dev
ARG UID=1000
ARG GID=1000
ARG IMG_NAME="moteus-dev"

# Check for mandatory build arguments
RUN : "${UID:?Build argument needs to be set and non-empty.}" && \
    : "${GID:?Build argument needs to be set and non-empty.}" && \
    : "${USERNAME}:?Build argument needs to be set and non-empty.}"

# Create the user
RUN groupadd --gid $GID $USERNAME \
    && useradd --uid $UID --gid $GID -m $USERNAME \
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# dialout group
RUN usermod -a -G dialout $USERNAME

# Install necessary packages
RUN apt-get install -y \
    mc \
    tree \
    git \
    wget \
    time \
    curl \
    gnupg \
    && rm -rf /var/lib/apt/lists/*

# Add Bazel's official repository and install Bazel
RUN apt-get update && apt-get install -y apt-transport-https curl gnupg \
    && curl -fsSL https://bazel.build/bazel-release.pub.gpg | gpg --dearmor >bazel-archive-keyring.gpg \
    && mv bazel-archive-keyring.gpg /usr/share/keyrings/bazel-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/bazel-archive-keyring.gpg] https://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list \
    && apt-get update && apt-get install -y bazel \
    && rm -rf /var/lib/apt/lists/*

# Create workspace
RUN mkdir -p /workspace && chown -R ${USERNAME} /workspace

# Setup folders for saving VSCode extensions
RUN mkdir -p /home/$USERNAME/.vscode-server/extensions \
    /home/$USERNAME/.vscode-server-insiders/extensions \
    && chown -R $USERNAME \
    /home/$USERNAME/.vscode-server \
    /home/$USERNAME/.vscode-server-insiders

# Set user
USER ${USERNAME}
RUN echo 'export PS1="🥝  \[\033[1;36m\]'"${IMG_NAME}"' \[\e[33m\]\W\[\e[m\] \[\033[1;36m\]# \[\033[0m\]"' >> ~/.bashrc

# Add Bazel binary to the PATH
ENV PATH="${PATH}:/home/${USERNAME}/.local/bin"

# Label image
USER root
WORKDIR /build
COPY Dockerfile Dockerfile

# Build timestamp
RUN echo ${IMG_NAME} >> build_date.txt && \
    date >> build_date.txt

USER ${USERNAME}
WORKDIR /workspace
