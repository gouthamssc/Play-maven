FROM openjdk:8
ENV PLAY_HOME="/opt/play"
RUN mkdir ${PLAY_HOME}
WORKDIR ${PLAY_HOME}
ADD my-application-play/target /opt/temp
RUN set -x && unzip -d /opt/svc /opt/temp/*.zip && mv /opt/svc/*/* ${PLAY_HOME}/ && rm ${PLAY_HOME}/*.bat 
RUN rm -r /opt/temp && rm -r /opt/svc 
RUN chown -R 185:0 /opt && chmod -R 774 /opt
USER 185
EXPOSE 9000 9443
CMD ${PLAY_HOME}/start -Dhttps.port=9443
