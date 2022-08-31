FROM node:latest

WORKDIR /app

# copy source
COPY . .

RUN npm install -g bower
RUN npm install -g http-server
RUN bower install --allow-root


# start server
EXPOSE 8443
STOPSIGNAL SIGTERM
CMD http-server -p 8443 -a 0.0.0.0 -s
