FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
# Instalamos todas las dependencias
RUN npm install
# Copiamos el resto del código
COPY . .

FROM node:18-alpine
WORKDIR /app
# Copiamos solo lo necesario desde la etapa de construcción
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
COPY --from=builder /app/server.js ./

EXPOSE 3001
CMD ["node", "server.js"]