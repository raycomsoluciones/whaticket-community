FROM node:18-bullseye as build-deps

# Instalación de utilidades básicas
RUN apt-get update && apt-get install -y wget gnupg

# Instalación de Dockerize
ENV DOCKERIZE_VERSION v0.6.1
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

# Instalación de Google Chrome estable y dependencias de Puppeteer
RUN apt-get update \
    && apt-get install -y wget gnupg \
    && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
    && apt-get update \
    && apt-get install -y google-chrome-stable=120.0.6099.109-1 \
       fonts-ipafont-gothic fonts-wqy-zenhei fonts-thai-tlwg fonts-kacst fonts-freefont-ttf libxss1 \
       xvfb libnss3 libxss1 libasound2 libatk1.0-0 libatk-bridge2.0-0 libgtk-3-0 libdrm2 \
       --no-install-recommends \
    && apt-mark hold google-chrome-stable \
    && rm -rf /var/lib/apt/lists/*

# Instalación de dumb-init
ADD https://github.com/Yelp/dumb-init/releases/download/v1.2.1/dumb-init_1.2.1_amd64 /usr/local/bin/dumb-init
RUN chmod +x /usr/local/bin/dumb-init

# Evitar descarga de Chromium
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true

# Directorio de trabajo
WORKDIR /usr/src/app

# Instalación de dependencias (solo una vez)
COPY package*.json ./
RUN npm install --legacy-peer-deps

# Copia del código y build
COPY . .
RUN npm run build

# Variables de entorno
ENV NODE_ENV=production
ENV PORT=3000
ENV CHROME_BIN=/usr/bin/google-chrome-stable
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/google-chrome-stable
ENV PUPPETEER_ARGS="--no-sandbox --disable-setuid-sandbox"
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true

# Exposición del puerto
EXPOSE 3000

# Ejecución final
ENTRYPOINT ["dumb-init", "--"]
CMD dockerize -wait tcp://${DB_HOST}:3306 \
  && npx sequelize db:migrate \
  && node dist/server.js
