FROM node:18-bullseye as build-deps

# 1. Instalar utilidades básicas y TODAS las dependencias de sistema para Puppeteer
RUN apt-get update && apt-get install -y \
    wget \
    gnupg \
    xvfb \
    libnss3 \
    libxss1 \
    libasound2 \
    libatk1.0-0 \
    libatk-bridge2.0-0 \
    libgtk-3-0 \
    libdrm2 \
    libgbm1 \
    libpangocairo-1.0-0 \
    libx11-xcb1 \
    libxcomposite1 \
    libxcursor1 \
    libxdamage1 \
    libxext6 \
    libxfixes3 \
    libxi6 \
    libxrandr2 \
    libxrender1 \
    libxtst6 \
    ca-certificates \
    fonts-liberation \
    libappindicator1 \
    lsb-release \
    xdg-utils \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# 2. Instalar Dockerize
ENV DOCKERIZE_VERSION v0.6.1
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

# 3. Instalar dumb-init
ADD https://github.com/Yelp/dumb-init/releases/download/v1.2.1/dumb-init_1.2.1_amd64 /usr/local/bin/dumb-init
RUN chmod +x /usr/local/bin/dumb-init

# 4. Directorio de trabajo
WORKDIR /usr/src/app

# 5. Instalación de dependencias
# (¡YA NO saltamos la descarga de Chromium!)
COPY package*.json ./
RUN npm install --legacy-peer-deps

# 6. Copia del código y build
COPY . .
RUN npm run build

# 7. Variables de entorno (¡YA NO necesitamos CHROME_BIN ni EXECUTABLE_PATH!)
ENV NODE_ENV=production
ENV PORT=3000
ENV PUPPETEER_ARGS="--no-sandbox --disable-setuid-sandbox"

# 8. Exposición del puerto
EXPOSE 3000

# 9. Ejecución final
ENTRYPOINT ["dumb-init", "--"]
CMD dockerize -wait tcp://${DB_HOST}:3306 \
  && npx sequelize db:migrate \
  && node dist/server.js
