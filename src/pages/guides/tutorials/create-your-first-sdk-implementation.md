---
title: Create Your First Firefly SDK Implementation - Adobe Firefly API
description: This how-to guides you through the process of integrating Adobe's Firefly workflows into your applications.
keywords:
  - Adobe Firefly Services
  - Firefly API
  - Integrating Firefly Services
  - Developer documentation
  - How-to guides
  - Tutorial
  - Firefly tutorial
  - Firefly API tutorial
  - Get started
  - Environment setup
  - Setup
  - Workflow
  - Credentials
  - Authentication
  - Sample code
  - Generate image
  - Call firefly
  - Sample call
  - Application development
  - First Firefly application
  - Firefly implementation
  - Prompts
  - Getting started with Firefly API
  - Application creation tutorial
  - Step-by-step guide
  - Development workflow
  - Application setup
  - API integration
  - User interface development
  - Backend implementation
  - Frontend development
  - SDK usage
  - Code examples
  - Automate processing
  - Development environment
  - Application deployment
  - Testing and debugging
  - Styles
contributors:
  - https://github.com/nimithajalal
  - https://github.com/cfjedimaster
hideBreadcrumbNav: true
---
# Create Your First Firefly SDK Implementation

As a developer, you have the flexibility to choose how you want to integrate with Firefly Services, tailoring the experience to your specific needs and preferences. 

## Overview

Whether you prefer the direct control and language-specific implementations provided by [REST APIs](../../firefly-api/guides/api/image_generation/) or the simplified, streamlined approach offered by our [Node SDK](https://developer.adobe.com/firefly-services/docs/guides/sdks/), Firefly Services has you covered.

## Choosing the Right Approach for Your Project

* __When to Use REST APIs__: If you need maximum flexibility, detailed control, and are working in a language other than JavaScript/TypeScript, direct REST API access might be the best choice for you.

* __When to Use the Node SDK__: If you're working in a `Node.js` environment and want to simplify the integration process, reduce boilerplate code, and speed up development, the Node SDK is the way to go.
By offering both direct REST API access and a Node SDK, Firefly Services ensures that you have the tools you need to build powerful integrations in the way that best suits your workflow. 

Let's explore what you need to get started with your first integration.

The SDK supports Node.js and either JavaScript or TypeScript. While this article will focus on Firefly APIs, the SDK also includes Photoshop and Lightroom APIs.

## Pre-requisites

* You will need a set of credentials for Firefly Services. You can get those [here](../get-started.md).

## The Workflow

Before digging into the code, let's break down the process.

1. Add the SDK packages
2. Setting up authentication
3. Instantiate Firefly
4. Generating an Image with a Prompt
5. Download the Results


## Step 1: Add the SDK packages

Let's begin by initializing a new `package.json` in your prompt with `npm init -y`. 

Next, we need to add the SDK. As described in the SDK's [readme](https://github.com/Firefly-Services/firefly-services-sdk-js), there are four individual packages you *can* install:

* The `common APIs` package is required for authentication, you'll always need this.
* A package for Firefly APIs.
* A package for Photoshop APIs.
* A package for Lightroom APIs.

For our needs, we only require the common and Firefly APIs. Install them like so:

```bash
npm install @adobe/firefly-services/common-apis
npm install @adobe/firefly-services/firefly-apis
```

Lastly, in order to use top-level await and imports, add a `type` setting of `module` to your package.json. Here's a complete example for reference:

```json
{
  "name": "code",
  "version": "1.0.0",
  "type":"module",
  "description": "",
  "main": "firefly.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@adobe/firefly-apis": "^1.0.0",
    "@adobe/firefly-services-common-apis": "^1.0.0"
  }
}
```

## Step 2: Setting up Authentication

Let's begin with a simple example, generating an image from a prompt. Our code needs to begin with authentication. Currently, authentication requires creating a utility function that returns an "auth provider" object. This is a bit complex, but can also be taken directly from the SDK's documentation.

```js
import { ServerToServerTokenProvider } from '@adobe/firefly-services-common-apis';

function getAuthProvider(clientId, clientSecret, scopes) {
    const serverToServerAuthDetails = { 
        clientId,
        clientSecret,
        scopes 
    };
    const serverToServerAuthOptions = {
        autoRefresh: true
    }
    return new ServerToServerTokenProvider(serverToServerAuthDetails, serverToServerAuthOptions);
}

// create auth config
const authProvider = getAuthProvider(process.env.CLIENT_ID, process.env.CLIENT_SECRET, 'openid,AdobeID,firefly_enterprise,firefly_api,ff_apis'); 
const config = {
    tokenProvider: authProvider,
    clientId: process.env.CLIENT_ID 
};
```

In the code above, we've created an `authProvider` object using our client ID and secret values as defined in two environment variables, `CLIENT_ID` and `CLIENT_SECRET`. This is wrapped up in a new object, `config`, that includes the `authProvider` as well as the client ID values.

## Step 3: Instantiate Firefly

The next part is much more simple, creating an instance of the Firefly SDK:

```js
import { FireflyClient } from '@adobe/firefly-apis';

const firefly = new FireflyClient(config);
```

## Step 4: Generating an Image with a Prompt

With authentication done and the Firefly client created, how do we generate images for a prompt? It takes all of one line! 

```js
const resp = await firefly.generateImages({prompt:'a cat riding a unicorn headed into the sunset, dramatic pose', n:4});
```

As a reminder, the Firefly API can accept *many* parameters, and they're all supported by the SDK, but in this case, we've passed just a prompt and the number of images required. The result of the SDK call is twofold - first a `result` JSON object that matches what the REST API returns, and secondly a set of `headers` you can inspect if needed.

Here's the JSON returned in the `result` key:

```json
{
	"version": "2.10.3",
	"size": {
		"width": 2048,
		"height": 2048
	},
	"predictedContentClass": "art",
	"outputs": [
		{
			"seed": 312577738,
			"image": {
				"id": "752f756f-f3b0-44d5-9ca5-c805628813fe",
				"presignedUrl": "https://pre-signed-firefly-prod.s3.amazonaws.com/images/752f756f-f3b0-44d5-9ca5-c805628813fe?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIARDA3TX66CSNORXF4%2F20240429%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240429T170605Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=7a02b554651d28e1971019de0cdc263ada189ea1bc7d7c7da7879517b1e75723"
			}
		},
		{
			"seed": 1089286377,
			"image": {
				"id": "ee062ccc-45ae-4d17-a6e5-a281a3b69d3f",
				"presignedUrl": "https://pre-signed-firefly-prod.s3.amazonaws.com/images/ee062ccc-45ae-4d17-a6e5-a281a3b69d3f?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIARDA3TX66CSNORXF4%2F20240429%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240429T170605Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=ef155784ffdfa0509d634681fcaf7fe670d1739029055d9f916d7865d3794478"
			}
		},
		{
			"seed": 2002836438,
			"image": {
				"id": "4936ff55-8c8f-489c-be95-ff0e00bf75bd",
				"presignedUrl": "https://pre-signed-firefly-prod.s3.amazonaws.com/images/4936ff55-8c8f-489c-be95-ff0e00bf75bd?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIARDA3TX66CSNORXF4%2F20240429%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240429T170605Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=a6053959373cbcf5e471658b042d334ebed352e55ff5d932e8301e8fca78fb5e"
			}
		},
		{
			"seed": 1011733134,
			"image": {
				"id": "62daaf9a-08ec-45f0-9985-573635c371b0",
				"presignedUrl": "https://pre-signed-firefly-prod.s3.amazonaws.com/images/62daaf9a-08ec-45f0-9985-573635c371b0?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIARDA3TX66CSNORXF4%2F20240429%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240429T170605Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=ae5635a67c7a32ba52ea3e2015242d430dce70e2de8bbb4e840614970cacf84a"
			}
		}
	]
}
```

## Step 5: Download the Results

Currently, the SDK doesn't support downloading the images for you, but this can be done with a few lines of code. Here's the required import and a basic utility function to download a URL to a path:

```js
import fs from 'fs';
import { Readable } from 'stream';
import { finished } from 'stream/promises';

async function downloadFile(url, filePath) {
	let res = await fetch(url);
	const body = Readable.fromWeb(res.body);
	const download_write_stream = fs.createWriteStream(filePath);
	return await finished(body.pipe(download_write_stream));
}
```

To put it all together, here's a complete script that handles the authentication, generating images based on our prompt, and then saving each result to the file system using the results `seed` value.

```js
import fs from 'fs';
import { Readable } from 'stream';
import { finished } from 'stream/promises';

import { FireflyClient } from '@adobe/firefly-apis';
import { ServerToServerTokenProvider } from '@adobe/firefly-services-common-apis';

async function downloadFile(url, filePath) {
	let res = await fetch(url);
	const body = Readable.fromWeb(res.body);
	const download_write_stream = fs.createWriteStream(filePath);
	return await finished(body.pipe(download_write_stream));
}

function getAuthProvider(clientId, clientSecret, scopes) {
    const serverToServerAuthDetails = { 
        clientId,
        clientSecret,
        scopes 
    };
    const serverToServerAuthOptions = {
        autoRefresh: true
    }
    return new ServerToServerTokenProvider(serverToServerAuthDetails, serverToServerAuthOptions);
}

// create auth config
const authProvider = getAuthProvider(process.env.CLIENT_ID, process.env.CLIENT_SECRET, 'openid,AdobeID,firefly_enterprise,firefly_api,ff_apis'); 
const config = {
    tokenProvider: authProvider,
    clientId: process.env.CLIENT_ID 
};

const firefly = new FireflyClient(config);

const resp = await firefly.generateImages({prompt:'a cat riding a unicorn headed into the sunset, dramatic pose', n:4});

for(let output of resp.result.outputs) {
	let fileName = `./${output.seed}.jpg`;
	await downloadFile(output.image.presignedUrl, fileName);
}
```

And here's one of the sample results of our prompt:

![Generated image](..//images/shot1.jpg)

## Additional workflow: Working with Reference Images

One of the benefits of having a Firefly SDK is how easy it is to build more complex workflows. To demonstrate a simple example of this, let's enhance our simple text-to-image prompt example by using a reference image. Reference images help guide the visual style of the generated result. 

In order for this to work, we first need our reference image. We'll use this:

[Source image](..//images/source_image.jpg)

Skipping over the authentication bits we've already covered, the first change is to upload our reference image:

```js
const uploadResp = await firefly.upload(new Blob([await fs.readFile('./source_image.jpg')],{type:'image/jpeg'}));
```

The `upload` SDK method requires a Blob which we've used wrapped around a file read on `./source_image.jpg`. As with the previous SDK example, the response includes a JSON result as well as the headers, with the JSON matching what you get when using the REST API. As an example:

```json
{
	"images": [
		{
		"id": "9f54159b-f0e9-4696-b4f5-f543a3fb90c0"
		}
	]
}
```

Using that reference when generating images looks like so:

```js
const resp = await firefly.generateImages({
    prompt:'a cat riding a unicorn headed into the sunset, dramatic pose', 
    styles: {
        referenceImage: { id:uploadResp.result.images[0].id } 
    },
	n:4
 });
 ```

 Compared to the previous version, we've added the `styles` attribute and passed in the `referenceImage` value. The changes to the generated images are immediately recognizable as being based on the style of the reference image. 

 ![Second generated image](../images/shot2.jpg)

## Next Steps

Now that you've seen how the SDK can *greatly* simplify your Firefly Services workflows, take a look over that [Github repository](https://github.com/Firefly-Services/firefly-services-sdk-js) and check out the docs and other samples as well.