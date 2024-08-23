# Use an Alpine Linux-based Node.js image as the base
FROM node:20-alpine as builder

# Set the working directory to /app
WORKDIR /app

RUN apk add --no-cache libc6-compat
RUN npm install -g pnpm

# Copy package.json and package-lock.json
COPY pnpm-lock.yaml ./

RUN pnpm fetch

# Copy the rest of the application code
COPY . .

# Build the Next.js app
RUN pnpm install
RUN pnpm run build

# Use a smaller base image for the final image
FROM node:20-alpine

# Set the working directory
WORKDIR /app

RUN apk add --no-cache libc6-compat
RUN npm install -g pnpm

# Copy the built Next.js app from the builder stage
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/pnpm-lock.yaml ./
COPY --from=builder /app/.next /app/.next
COPY --from=builder /app/public /app/public

RUN pnpm install --prod

# Expose the port used by Next.js
EXPOSE 3000

# Set the command to run the Next.js app
CMD ["pnpm", "start"]