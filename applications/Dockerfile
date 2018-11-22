FROM risingstack/alpine:3.3-v4.2.6-1.1.3
MAINTAINER letters2hamza@gmail.com
ADD package.json package.json
RUN npm install
ADD server.js server.js
EXPOSE 3000
CMD ["npm", "start"]