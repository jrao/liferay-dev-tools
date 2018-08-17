# Elasticsearch 6 with plugins used by Liferay
FROM docker.elastic.co/elasticsearch/elasticsearch:6.1.3

# Install elasticsearch plugins
RUN bin/elasticsearch-plugin install --batch analysis-icu
RUN bin/elasticsearch-plugin install --batch analysis-kuromoji
RUN bin/elasticsearch-plugin install --batch analysis-smartcn
RUN bin/elasticsearch-plugin install --batch analysis-stempel