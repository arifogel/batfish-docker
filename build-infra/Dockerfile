# Setting the parent image to ubuntu
FROM ubuntu:16.04
LABEL vendor="The Batfish Open Source Project"

# Set a Working Dir
WORKDIR /root/workdir

# Installing Java, python, z3 dependencies
RUN apt-get update && apt-get install -y \
    apt-transport-https \
    binutils \
    ca-certificates \
    curl \
    gcc \
    git \
    jq \
    lsb-release \
    net-tools \
    openjdk-8-jdk \
    python-minimal \
    python3-dev \
    software-properties-common \
    unzip \
    wget \
&&  rm -rf /var/lib/apt/lists/* \
    /var/cache/oracle-jdk8-installer \
&&  apt-get clean

# Install pip
RUN curl https://bootstrap.pypa.io/get-pip.py | python3

# Install virtualenv
RUN python3 -m pip install virtualenv

# Create AWS-CLI virtualenv
RUN python3 -m virtualenv .venv-aws && . .venv-aws/bin/activate && pip install awscli && deactivate

# Install docker
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
RUN add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
&& apt-get update \
&& apt-get install -y docker-ce \
&& rm -rf /var/lib/apt/lists/* \
&& apt-get clean

# Install Z3
RUN wget --no-verbose https://raw.githubusercontent.com/batfish/batfish/master/tools/install_z3.sh \
&& bash install_z3.sh \
&& rm -r ~/.batfish_z3_cache \
&& rm install_z3.sh

# Install bazel
RUN wget --no-verbose https://github.com/bazelbuild/bazel/releases/download/0.19.2/bazel-0.19.2-installer-linux-x86_64.sh -O install_bazel.sh \
&& chmod +x install_bazel.sh \
&& ./install_bazel.sh --user \
&& rm -f install_bazel.sh

# Install maven
RUN wget --no-verbose https://www-us.apache.org/dist/maven/maven-3/3.6.0/binaries/apache-maven-3.6.0-bin.zip \
&& unzip apache-maven-3.6.0-bin.zip \
&& rm -f apache-maven-3.6.0-bin.zip

# Setup JAVA_HOME
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64/
# Set Java to obey cgroup limits
ENV _JAVA_OPTIONS -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1

# Add Bazel and Maven to PATH
ENV PATH $PATH:/root/workdir/apache-maven-3.6.0/bin:/root/bin

# Fetch all the current Maven and bazel dependencies
ADD .bazelrc /root
RUN git clone --depth=1 --branch=circle-ci https://github.com/dhalperi/batfish \
&& cd batfish \
&& mvn -f projects verify -DskipTests=false \
&& mvn dependency:get -Dartifact=com.google.googlejavaformat:google-java-format:1.6:jar:all-deps \
&& cd .. \
&& rm -rf batfish

ENV _JAVA_OPTIONS -Xmx2g
