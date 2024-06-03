# Need to create a Dockerfile for java based applications
# Dockerfile for Eureka
FROM openjdk:11
ARG JAR_SOURCE
ENV JAR_SOURCE ${JAR_SOURCE}
ARG JAR_DEST
ENV JAR_DEST ${JAR_DEST}
RUN mkdir -p /opt/sureshindrala1
WORKDIR /opt/sureshindrala1
COPY ["${JAR_SOURCE}","/opt/sureshindrala1/i27cart-eureka.jar"]
RUN chmod 777 /opt/sureshindrala1
EXPOSE 8761
CMD ["java","jar","/opt/sureshindrala1/i27cart-eureka.jar"]