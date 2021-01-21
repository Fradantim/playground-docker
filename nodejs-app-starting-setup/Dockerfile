FROM node:14

WORKDIR /app

# The second dot is actually /app (WORKDIR)
# COPY . .
# Both of these instructions work equally
# COPY . /app

COPY package.json /app

RUN npm install

# This es where most of our changes will exists, 
# this way the previous instructions will keep 
# using the same layers, and from here onward 
# only this layer keeps evolving
COPY . /app

EXPOSE 80

CMD [ "node", "server.js" ]