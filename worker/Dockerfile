#FROM k8s-openjdk-jdk:1.8.0_77
FROM java:7

RUN sed --in-place 's/httpredir.debian.org/ftp.th.debian.org/' /etc/apt/sources.list
RUN apt-get update -qq && apt-get install -y maven && apt-get clean

#RUN MAVEN_VERSION=3.3.3 \
# && cd /usr/share \
# && wget http://archive.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz -O - | tar xzf - \
# && mv /usr/share/apache-maven-$MAVEN_VERSION /usr/share/maven \
# && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

WORKDIR /code

ADD pom.xml /code/pom.xml
RUN ["mvn", "dependency:resolve"]
RUN ["mvn", "verify"]

# Adding source, compile and package into a fat jar
ADD src /code/src
RUN ["mvn", "package"]

#CMD ["/usr/lib/jvm/java-7-openjdk-amd64/bin/java", "-jar", "target/worker-jar-with-dependencies.jar"]

RUN echo 'hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4' >> /etc/nsswitch.conf

CMD ["java", "-jar", "target/worker-jar-with-dependencies.jar"]
