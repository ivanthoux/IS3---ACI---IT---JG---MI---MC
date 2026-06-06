FROM node:20-alpine

RUN apk add --no-cache \
    python3 \
    make \
    g++ \
    cairo-dev \
    pango-dev \
    libjpeg-turbo-dev \
    giflib-dev


WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN mkdir -p public/certificates public/qrs public/reports
EXPOSE 3000
CMD ["node", "src/app.js"]