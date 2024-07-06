# Use consistent capitalization for FROM and AS
FROM node:18-bookworm-slim AS global-deps-stage

WORKDIR /app

# Installing necessary packages and dependencies
RUN apt-get update \
  && apt-get install -y wget gnupg nano ffmpeg=7:5.1.4-0+deb12u1 \
  && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
  && sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
  && apt-get update \
  && apt-get install -y google-chrome-stable fonts-ipafont-gothic fonts-wqy-zenhei fonts-thai-tlwg fonts-kacst fonts-freefont-ttf libxss1 \
  --no-install-recommends \
  && rm -rf /var/lib/apt/lists/* && npm install pm2@latest -g

# Environment variables
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
ENV NODE_ENV=dev
ENV CHROME_BIN=google-chrome-stable

FROM global-deps-stage AS develop-stage
COPY package*.json ./
RUN npm install

FROM develop-stage AS build-stage
COPY . .
RUN npm run build

FROM build-stage AS development-stage
ENV NODE_ENV=development
ENTRYPOINT ["npm", "run", "dev:server"]

FROM build-stage AS production-stage
ENV NODE_ENV=production

# Ensure docker-entrypoint.sh exists and is in the correct directory
# Make sure this file is in the project root or adjust the path accordingly
COPY docker-entrypoint.sh /app/docker-entrypoint.sh
RUN chmod +x /app/docker-entrypoint.sh
ENTRYPOINT ["/app/docker-entrypoint.sh"]
