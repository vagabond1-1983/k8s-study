#Refer: https://github.com/newtmitch/docker-sonar-scanner/blob/master/Dockerfile.sonarscanner-4.0.0-full
FROM maven:3.6.3-jdk-8-openj9

RUN apt-get update
RUN apt-get install -y curl git tmux htop sudo unzip

WORKDIR /usr/src

# 当需要使用公司内部的maven settings.xml和配置全局的sonar-scanner.properties时，可以编辑这两个文件
ADD ./settings.xml settings.xml
ADD ./sonar-scanner.properties sonar-scanner.properties

RUN curl --insecure -o ./sonarscanner.zip -L https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.0.0.1744-linux.zip && \
	unzip sonarscanner.zip && \
	rm sonarscanner.zip && \
	mv sonar-scanner-4.0.0.1744-linux /usr/lib/sonar-scanner && \
	ln -s /usr/lib/sonar-scanner/bin/sonar-scanner /usr/local/bin/sonar-scanner && \
	cp settings.xml ${MAVEN_HOME}/conf && \
	cp sonar-scanner.properties /usr/lib/sonar-scanner/conf

ENV SONAR_RUNNER_HOME=/usr/lib/sonar-scanner

# Separating ENTRYPOINT and CMD operations allows for core execution variables to
# be easily overridden by passing them in as part of the `docker run` command.
# This allows the default /usr/src base dir to be overridden by users as-needed.
ENTRYPOINT ["sonar-scanner"] 
CMD ["-Dsonar.projectBaseDir=/usr/src"]