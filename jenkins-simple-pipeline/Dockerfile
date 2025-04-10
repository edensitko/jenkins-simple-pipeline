FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --prefer-offline --no-audit
COPY . .
CMD ["npm", "start"]
