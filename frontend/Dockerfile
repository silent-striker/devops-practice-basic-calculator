FROM node:bullseye-slim

WORKDIR /app/

COPY package.json .

COPY /public/ ./public/

COPY /src/ ./src/

ENV REACT_APP_BACKEND_URL="http://localhost"
ENV REACT_APP_BACKEND_PORT=10000

RUN npm install

RUN npm install -g serve

RUN npm run build

ENV SERVER_PORT=3000
EXPOSE ${SERVER_PORT}

# CMD ["npm", "start"]
CMD ["sh", "-c", "serve -s build -l ${SERVER_PORT}"]