FROM node:8
RUN useradd -m -u 5000 go
#WORKDIR /home/go
#COPY ./package.json /home/go/package.json
RUN runuser -l go -c 'ls -lah'
RUN npm cache clean --force
RUN npm install -g yarn sass gulp karma

# The user id is specied in the Go-Agent login information, currently it is 5K


