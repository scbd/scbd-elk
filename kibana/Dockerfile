# https://github.com/elastic/kibana-docker
FROM docker.elastic.co/kibana/kibana:7.1.1
# blaisefonseca/kibana:7.0.0

# Add your kibana plugins setup here
# Example: RUN kibana-plugin install <name|url>

COPY ./kibana/plugins/logtrail/logtrail-7.1.1-0.1.31.zip /plugins/

RUN kibana-plugin install file:///plugins/logtrail-7.1.1-0.1.31.zip
