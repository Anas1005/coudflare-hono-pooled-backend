# Cloudflare with Hono and Prisma Accelerator Setup Guide

This guide will walk you through the process of setting up Cloudflare with Hono and Prisma Accelerator as a pooled connection. Prisma Accelerator provides additional functionality for Prisma, including connection pooling, which ensures that multiple workers or threads in your application can efficiently share and reuse a pool of pre-established database connections. This reduces resource consumption and improves performance.

**Notes:**
- Cloudflare Workers allow environment variables in the wrangler.toml file through built-in support for wrangler secret management. It's important to note that the CLI picks up these variables from the .env file, not the backend.
- When developing npm packages:
  1. Organize common functionalities in a shared module or folder.
  2. Use Zod type inference inside the common module to export common services.
  3. Publish the module to npm for easy consumption by other projects.
  4. Set "declaration" to true in tsconfig.json to generate .d.ts files.
  5. Ensure not to include .ts files in the published package, only include .js and .d.ts files.

## Prerequisites


Before proceeding, ensure you have the following installed:

- Node.js and npm

## Setup Steps

1. **Install Hono:**
    - Hono is a Node.js package that provides tools for creating HTTP/HTTPS servers with ease. It simplifies the process of creating and managing servers, making it ideal for building web applications.
    ```bash
    npm create hono@latest
    ```

2. **Install Prisma:**
    - Prisma is a modern database toolkit that simplifies database access for Node.js and TypeScript applications. It provides an intuitive and type-safe way to interact with databases, making database operations more efficient and less error-prone.
    ```bash
    npm i prisma
    ```

3. **Initialize Prisma:**
    - Prisma init command initializes a new Prisma project in the current directory. It creates the necessary directory structure and configuration files for working with Prisma.
    ```bash
    npx prisma init
    ```

4. **Hardcode the URL in Schema.prisma:**
    - In the Schema.prisma file, you hardcode the database URL that Prisma will use to connect to your database. This URL contains the necessary connection information, such as the database host, port, username, password, and database name.
  
6. **Run the migration command**
     ```bash
    npx prisma migrate dev --name "Initialised Schema"
    ```

7. **Set DIRECT_URL in .env:**
    - The DIRECT_URL environment variable specifies the direct database URL that Prisma will use for connections. This URL is used when establishing direct connections to the database without using a connection pool.
    ```plaintext
    DIRECT_URL = "your_database_url"
    ```

8. **Set DATABASE_URL(Connection Pool String) in wrangler.toml:**
    - Update your wrangler.toml file with the DATABASE_URL variable, which defines the connection pool string for Prisma Accelerator.
    ```toml
    [vars]
    DATABASE_URL="your_connection_pool_string"
    ```

9. **Install Prisma Accelerator Extension:**
    - Prisma Accelerator Extension provides additional functionality for Prisma, including connection pooling and performance optimization.
    ```bash
    npm i @prisma/extension-accelerate
    ```

10. **Generate Prisma Client:**
    - Generate Prisma client code based on your schema definition file and database connection information. Since Cloudflare Workers have specific requirements for bundle optimization, it's necessary to include the `--no-engine` flag to ensure that bubdle is generated without any Prisma engine files.
    ```bash
    npx prisma generate --no-engine
    ```


11. **Update Schema.prisma:**
    - Update your Schema.prisma file with the necessary generator and datasource blocks to use Prisma Accelerator and the defined database URLs.
    ```prisma
    generator client {
      provider = "prisma-client-js"
    }

    datasource db {
      provider = "postgresql"
      url      = env("DATABASE_URL")
      directUrl = env("DIRECT_URL")
    }
    ```

12. **Run Development Server:**
    - Start the development server using the appropriate command specified by Hono. Refer to the Hono documentation for syntax and usage details.
    ```bash
    npm run dev
    ```
    Ensure to refer to the Hono documentation for syntax and usage details to mimic the express like functionalities and to write the code and logic for your Cloudflare 
    application.



**Deployment Steps:**

13. **Deploy to Cloudflare Workers:**
    - Before deploying, ensure you are logged in to Cloudflare using the Wrangler CLI. Run the following commands to log in and verify your credentials:
    ```bash
    npm run wrangler login
    npm run wrangler whoami
    ```
    - Once logged in, deploy your application to Cloudflare Workers using the following command:
    ```bash
    npm run deploy
    ```
    This command will build and deploy your application to Cloudflare's network. Ensure that your Wrangler configuration file (wrangler.toml) is correctly set up with the necessary settings for your deployment.



