FROM bde2020/spark-base:3.0.0-hadoop3.2

COPY master.sh /
RUN chown -R root:root /master.sh
RUN chmod u+x /master.sh

COPY hive-site.xml /spark/conf/

ENV SPARK_MASTER_PORT 7077
ENV SPARK_MASTER_WEBUI_PORT 8080
ENV SPARK_MASTER_LOG /spark/logs
ENV SPARK_HOME=/spark
ENV PATH=${SPARK_HOME}/bin:$PATH

EXPOSE 8080 7077 6066

CMD ["/bin/bash", "/master.sh"]
