# Elasticsearch 2 with plugins used by Liferay
FROM elasticsearch:2.4.4

# Install elasticsearch plugins
RUN bin/plugin install analysis-icu
RUN bin/plugin install analysis-kuromoji
RUN bin/plugin install analysis-smartcn
RUN bin/plugin install analysis-stempel

# Install shield plugins
RUN bin/plugin install license
RUN bin/plugin install shield