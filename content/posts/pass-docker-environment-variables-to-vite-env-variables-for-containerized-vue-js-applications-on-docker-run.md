---
author: "CodeGenos"
title: "How to pass Docker Environment Variables to Vite Env Variables for Containerized Vue.js Applications on Docker Run?"
date: 2023-09-13T23:43:16+03:00
description: "In this article i'll show you how to pass Docker Environment Variables to Vite Env Variables for Containerized Vue.js Applications on Docker Run"
tags: ["Docker", "Vite", "Vue.js"]
categories: ["Docker", "Vue.js", "Vite"]
ShowToc: false
ShowBreadCrumbs: true
draft: false
---

Environment variables can only be used in Vue.js applications at build time and the variables are hardcoded in javascript files during build. 
When you build docker image for the Vue.js application, you use the same image for `test` and `production` environments. 
But the Vue.js application config values can be different in `test` and `production` environments. For example: api url configuration value. 
Since the variables are different in those environments you must pass the configuration values for test and production environments at docker run:

```bash
docker run -d -p 8080:80 -e VARIABLE1=var1value -e VARIABLE2=var2value my-image
```

But the Vue.js configuration variables are hardcoded to javascript files in the docker image after build. We must find a way to substitute the values in javascript files when a container is started. I'll show you how to do that.

## Vue.js application
The Vue.js application uses Vite env variables. We have to variables: `VITE_VARIABLE1` and `VITE_VARIABLE2`.

We are using Vite.js build tool and şn Vite, environmental variables need to be prefixed with `VITE_` as in `VITE_*variable-name`.

### .env Files
In `.env.development` file we can hardcode the variables:
```
VITE_VARIABLE1=var1valuedev
VITE_VARIABLE2=var2valuedev
```

In `.env.production` file we give variable names in `__...__` format.
```
VITE_VARIABLE1=__VARIABLE1__
VITE_VARIABLE2=__VARIABLE2__
```

### config.ts
We create a separate `config.ts` file and read all all variables with `import.meta.env`.

```ts
export interface Config {
    variable1: string
    variable2: string
}

export const config: Config = {
    variable1: import.meta.env.VITE_VARIABLE1,
    variable2: import.meta.env.VITE_VARIABLE2
}
```

### Use config variables in application
We import the config from '../config' and use in vue template: 

```html
<script setup lang="ts">
import { config } from '../config'
</script>

<template>
  <div class="card">  
    <p>var1 = {{ config.variable1 }} </p>
    <p>var2 = {{ config.variable2 }} </p>
  </div>
</template>
```

### vite.config.ts
In `vite.config.ts` we create a separate output bundle for `config.js` using `manualChunks`. Because we will only substitute the variables in config.js file. We are doing this because we don't want to accidentally change the values that might have the same text as our variable names in other javascript files.

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  build: {
    rollupOptions: {
      output: {
        // Define the main output bundle (e.g., for your application)
        entryFileNames: '[name]-[hash].js',
        // Define the directory where the main bundle should be output
        dir: 'dist',
      },
      // Create a separate output bundle for your specific file
      manualChunks(id) {
        if (id.endsWith('src/config.ts')) {
          return 'config' // This will create a separate bundle for config-[hash].js
        }
      }
    }
  }
})
```

Using `manualChunks` we create a separate bundle for `src/config.ts`. After run `vite build`, a separate bundle for `config-[hash].js` file will be created. For example: `config-4f3cd5bb.js`.

### Substitution of variables after vite build
`__VARIABLE1__` and `__VARIABLE2__` comes from `.env.production` file.

`config-4f3cd5bb.js` file:
```js
const _={variable1:"__VARIABLE1__",variable2:"__VARIABLE2__"};export{_ as c};
```
After we serve the application with `npm run dev`. The page looks like this:
![Dev Page](/posts/images/vuejs-vite-env-variables-dev-html-page.jpg)

## DockerFile

In docker file we run `entrypoint.sh` as `ENTRYPOINT`. In `entrypoint.sh` we substitute variables and run `nginx`. 

We are substituting variables in `ENTRYPOINT` in order to read the environment variables passed at `docker run`. 

If we have used `RUN /entrypoint.sh` instead of `ENTRYPOINT ["/entrypoint.sh"]` we wouldn't be able to environment variables at `docker run`. 

Because `RUN` is used during image build time to execute commands and create image layers, while `ENTRYPOINT` is used to define the container's main process or command at runtime.

```DockerFile
# Build the Vue app
FROM node:18-alpine as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY ./ .
RUN npm run build

# Copy the built app in an NGINX container
FROM nginx as production-stage
RUN mkdir /app
COPY --from=build-stage /app/dist /app
COPY nginx.conf /etc/nginx/nginx.conf

# Copy entrypoint.sh
COPY ./entrypoint.sh /entrypoint.sh
# Make entrypoint.sh executable
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

### entrypoint.sh file
In `entrypoint.sh` we substitute variables and run `nginx`. 

In `keys` array we store our variables. 

We iterate the keys and get environment variable values with `value=$(eval echo "\$$key")`. 

And then replace the value using `sed` command.
`sed` command replaces `__VARIABLE1__` and `__VARIABLE2__` values with the environment variables in docker run: `docker run -d -p 8080:80 -e VARIABLE1=var1valueprod -e VARIABLE2=var2valueprod my-image`.

```bash
#!/bin/bash

ROOT_DIR=/app

keys=("VARIABLE1" "VARIABLE2")

# Replace env vars in files served by NGINX
for file in $ROOT_DIR/assets/config-*.js*;
do
  echo "Processing $file ...";
  for key in "${keys[@]}"
  do
    # Get environment variable
    value=$(eval echo "\$$key")
    echo "replace $key by $value"

    # replace __[variable_name]__ value with environment variable
    sed -i 's|__'"$key"'__|'"$value"'|g' $file
  done
done

nginx -g 'daemon off;'
```
## Build docker image and run docker container

Build image and run docker image:
```bash
docker build . -t vue-environment-variables-docker
docker run -d -p 8080:80 -e VARIABLE1=var1valueprod -e VARIABLE2=var2valueprod vue-environment-variables-docker
```

The final `config-4f3cd5bb.js` file:
```js
const _={variable1:"var1valueprod",variable2:"var2valueprod"};export{_ as c};
```

`http://localhost:8080/` page looks like this:
![Prod Page](/posts/images/vuejs-vite-env-variables-prod-html-page.jpg)

Thanks for reading.

Happy coding!

