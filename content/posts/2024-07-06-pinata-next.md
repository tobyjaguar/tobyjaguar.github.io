---
title: "Pinata and NextJS APIs"
date: 2024-07-06
---

## Pinning with Pinata and old busted NextJS APIs
![title](/assets/posts/pinata-next/_sail-boat-desert-midjourney.jpeg)
Photo by <a href="https://www.midjourney.com/jobs/13045c47-d8a9-41e2-8e18-9baec63cc530?index=0">Sebdunedin from Midjourney</a>

### Integrating NextJS Pages Router with Pinata

Well, it has certainly been some time since this ole place has been
updated. Probably because a company was paying me to not write blog posts about quirky software problems, and I couldn't write blog posts about their tech stack. Anywho, this blog post is about the nightmare of integrating the [Pinata](https://docs.pinata.cloud/pinning/pinning-files) pinning APIs with NextJS Pages Router APIs.

Why would we want to do this? Pinata is a pinning service for [IPFS](https://ipfs.tech/), the platform allows you to upload content to the network and receive a content identifier in return that can be used to locate the asset on the network later. It all sounds well and good until the guy that does the frontend on the app gets a fulltime gig, and you are left trying to make changes to an already build NextJS app. Then the pain comes!

### The Problem

"Hey, you wanna bulid an app?" 

[Ethergrams](https://ethergrams.xyz) already had a storage solution for its NFT content, [nft.storage](https://nft.storage/), except they decided to take their storage solution in a different direction, which totally hosed our app, because we didn't want to pay anything for storage costs, because no one uses the app anyway, and we aren't crypto billionaires, or millionaires, or thousandaires. We are in fact quite poor. 

So, your previous storage provider no longer offers your app a storage solution, and you attempt to integrate with their recommended partners, [Pinata](https://docs.pinata.cloud/pinning/pinning-files). Pinata has APIs and an SDK, woot! This is going to be easy. A piece of cake. A slam dunk tech switch out. A copy and paste solution. Except no, nothing works, and I hate everything.

### The Solution

Pinata has an SDK that cannot be used client-side because it requires a super secret API key that guards resources assigned to your pinata account. So, copying the SDK code from the docs into your previously build (not by you) NextJS app and using that is not recommended. And come to find out, you can't use that on your NextJS app (still not built by you) because the Pinata SDK requires the *fs* NodeJS module which isn't available client-side anyway. So we are left with the APIs, which is the preferred method anyway.

Great. The API examples are super simple, and will just be a copy pasta solution to switch out support for the new IPFS storage solution. Except it doesn't work. And it doesn't work for so many reasons. 

The code is rather simple. We need to upload user generated pictures to IPFS and then the NFT metadata, and then mint the EtherGram. So we want to upload an image. The code for such is provided as:

```javascript
const JWT = 'YOUR_PINATA_JWT';

async function pinFileToIPFS() {
  try {
    const text = "Hello World!";
    const blob = new Blob([text], { type: "text/plain" });
    const file = new File([blob], "hello-world.txt")
    const data = new FormData();
    data.append("file", file);

    const res = await fetch("https://api.pinata.cloud/pinning/pinFileToIPFS", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${JWT}`,
      },
      body: data,
    });
    const resData = await res.json();
    console.log(resData);
  } catch (error) {
    console.log(error);
  }
};

await pinFileToIPFS()
```
[Blob](https://docs.pinata.cloud/pinning/pinning-files#blob)

Okay, great. I'll take one of those and a metadata upload, and be on my way. Except for that little detail at the top, that JWT import. We can't be doing that on the client-side because that will reveal our super secret resource access token to whomever downloads the app in a browser. Oops, and that isn't object oriented programming, that's you just leaked your key, bad, no no.

Okay so the Pinata [Docs](https://docs.pinata.cloud/quickstart/next-js) have a quickstart for NextJS apps, and I read it, but didn't really read it, and now I am re-reading it because it is saying I need to create a Next API route which can then access the JWT server-side avoiding the exposer to the client in the browser.

### Next APIs

Well I don't understand what that means, but it shouldn't be too bad, right?

So NextJS has a built in concept of where execution happens, which is a little different from the React model where everything happens on the user's computer. NextJS allows for execution to happen on the client *and* the server. So the app can have execution happening on the server for some things (like accessing secrets), while other things happen on the user's computer, like the traditional app navigation and client interactions. 

What is being suggested in the Pinata docs is to create a process that handles the secret, and keep that on the server running the app. In this scenario the client application will call an internal Next API route on the server, and the server will answer the call with the payload the client sent. The example for the server side route is given in the docs as:

```javascript
import { NextResponse, NextRequest } from "next/server";

export const config = {
  api: {
    bodyParser: false,
  },
};

export async function POST(request: NextRequest) {
  try {
    const data = await request.formData();
    const file: File | null = data.get("file") as unknown as File;
    data.append("file", file);
    data.append("pinataMetadata", JSON.stringify({ name: "File to upload" }));
    const res = await fetch("https://api.pinata.cloud/pinning/pinFileToIPFS", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${process.env.PINATA_JWT}`,
      },
      body: data,
    });
    const { IpfsHash } = await res.json();
    console.log(IpfsHash);

    return NextResponse.json({ IpfsHash }, { status: 200 });
  } catch (e) {
    console.log(e);
    return NextResponse.json(
      { error: "Internal Server Error" },
      { status: 500 }
    );
  }
}
```
[route](https://docs.pinata.cloud/quickstart/next-js#create-api-route)

Okay, so we just copy and paste this in and it works. Cool.

If anyone has made it this far, what are you doing?! But here comes the good stuff. This design pattern is for NextJS App Router APIs. Which, from quite a bit of searching, seems to be how most people are building their Next APIs lately. Our app (the one not build by you) uses NextJS Pages Router, which just explodes everything apparently. 

For anyone using NextJS Pages Router APIs wanting to upload an image, the following details the solution I arrived at over a couple of days of toil and cursing.

### Pages Router is different

Okay, here is the goods. The Pager Router APIs are different from the App Router APIs, the latter of which have a lot of goodies baked into their request and response interfaces. The Pager Router API request and response does not. So that simple little line above in the example:

`const data = await request.formData();`

does **not** work. It really does not work. The hilarious thing about that line of code is that the entire internet just keeps suggesting that solution over and over again when anyone asks why their multi-part form data isn't parsing.

The long and short of it is that the Pinata docs suggest converting an image blob (if the image happens to be a blob, as was our case) to a File type and sending the file as multi-part form data to the Next API route, parse the form data back into the file, grab your JWT, and then reformat the file as form data, and finally call their API to upload the file/image. That all will work if the Next app is using the newer App Router handlers, and it utterly fails if the app is using the Pages Router APIs. It's getting a bit deep in the weeds, so I will share a solution, to help my future self and anyone else who has the same issue and finds their way here.

### What Worked

The import of the Pages Router API interface types are different as are their features. The imports to the API that will handle image uploads is as follows:

```javascript
import fs from "fs";
import type { NextApiRequest, NextApiResponse } from 'next';
import { Formidable } from 'formidable';
```
The Pages Router imports the `NextApiRequest` rather than the `NextRequest` which is imported from `next/server` on the App Router APIs. They do different things, and most improtantly for this topic, the `NextRequest` has the `.formData()` function available on it, where the `NextApiRequest` *does not*! We are going to need a form data parser for this. And that is there the formidable package comes in. There are a handful of packages that do this. Most posts seem to use formidable, so that is the solution here. Multiparty also would work.

Next, there are a couple ticks needed, yay!

```javascript
interface IFields {
  name: string[];
}

type ResponseData = {
  IpfsHash: string | null,
  error: string | null
}

export const config = {
  api: {
    bodyParser: false,
  },
};
```

The formidable package will parse out fields and files, so we want to type the fields. We also want to type the files, but that one gave me a lot of trouble, so it gets the `any` type. The smarter will type it as needed.

The ResponseData is boilerplate. 

And the trick needed is that you tell the API's internal configuration (yeah, that's a thing), to *not* parse the body. This is needed for reasons, as details on the interwebs.

The other big trick, which is explained *NOWHERE* is that you will need to get the image data from disk via the filepath on the file that was sent to the internal API. That function looks like this:

```javascript
const getDataFile = (files: any) => {
  const buffer = fs.readFileSync(files.filepath);
  const arraybuffer = Uint8Array.from(buffer);
  const file = new File([arraybuffer], files.originalFilename, {
    type: files.mimetype,
  });
  return file;
};
```
I do apologize, as I have lost the attribution for this little code snippet. It is sad, as the solution did not have any upvotes, but worked for me. :(

The `getDataFile()` function will pull the data from the tmp directory storing the file. 

*note: this will be really hilarious when I deploy the app and this solution that works locally on a dev server doesn't work on the deployed server*

Next is just a helper function to make the Pinata API call:

```javascript
async function uploadImage(name: string, file: File): Promise<ResponseData> {
  const data = new FormData();
  data.append("file", file);
  data.append("pinataMetadata", JSON.stringify({ name }));
  const res = await fetch("https://api.pinata.cloud/pinning/pinFileToIPFS", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${process.env.PINATA_JWT}`,
    },
    body: data,
  });
  const { IpfsHash } = await res.json();
  return { IpfsHash, error: null };
}
```
This is essentailly their boilerplate code converted to the Pages Router design pattern.

The last part is the route handler:

```javascript
export default async function handler(
  request: NextApiRequest,
  response: NextApiResponse<ResponseData>
  ) {
  try {

    const data: { fields: unknown, files: unknown } = await new Promise((resolve, reject) => {
      const form = new Formidable();
      form.parse(request, (err, fields, files) => {
        if (err) reject({ err });
        resolve({ fields, files });
      });
    });

    const fields = data.fields as IFields;
    const files = data.files as any;
    let fileName = fields.name[0] as string;
    let file = getDataFile(files.file[0]);
    const res = await uploadImage(fileName, file);
    return response.status(200).send(res);
  } catch (error) {
    console.log(error);
    return response.status(500).send({ 
      IpfsHash: null,
      error: "Internal Server Error" 
    });
  }
}
```

The handler will return the ResponseData type declared above, which has an IPFS CID which is returned from the third-party API, and an optional error message. There were also many design patterns that used `const form = new formidable.IncomingForm();` and also an async pattern:

```javascript
form.parse(req, async (err: any, fields: any, files: any) => {
  if (err) {
      console.error(err);
      res.status(500).json({ message: 'Error parsing the form data.' });
      return;
  }
...
```
Choose your own adventure here.

The main sticking points have to do with the Pages Router working differently than the App Router APIs, specifically the lack of form data parsing. The next trick is *not* parsing the body data of the request, which is detailed in the config. Lastly the biggest trick, in my opinion, is getting the data back from the request. This is easily done in the documentation code by calling:

```javascript
const data = await request.formData();
const file: File | null = data.get("file") as unknown as File;
```

None of that works for the Pages Router API request. The `.formData()` function is not available and once the request is parsed with the formidable package, an additional step of "getting" the data is needed, here it is accessed via the filesystem module and read into a buffer:

```javascript
const buffer = fs.readFileSync(files.filepath);
```
And that is it. Pretty easy copy pasta. Except I couldn't find a full working example of this on all of the internet. So here is my little contribution:

```javascript
import fs from "fs";
import type { NextApiRequest, NextApiResponse } from 'next';
import { Formidable } from 'formidable';

interface IFields {
  name: string[];
}

type ResponseData = {
  IpfsHash: string | null,
  error: string | null
}

export const config = {
  api: {
    bodyParser: false,
  },
};

const getDataFile = (files: any) => {
  const buffer = fs.readFileSync(files.filepath);
  const arraybuffer = Uint8Array.from(buffer);
  const file = new File([arraybuffer], files.originalFilename, {
    type: files.mimetype,
  });
  return file;
};

async function uploadImage(name: string, file: File): Promise<ResponseData> {
  const data = new FormData();
  data.append("file", file);
  data.append("pinataMetadata", JSON.stringify({ name }));
  const res = await fetch("https://api.pinata.cloud/pinning/pinFileToIPFS", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${process.env.PINATA_JWT}`,
    },
    body: data,
  });
  const { IpfsHash } = await res.json();
  return { IpfsHash, error: null };
}

export default async function handler(
  request: NextApiRequest,
  response: NextApiResponse<ResponseData>
  ) {
  try {

    const data: { fields: unknown, files: unknown } = await new Promise((resolve, reject) => {
      const form = new Formidable();
      form.parse(request, (err, fields, files) => {
        if (err) reject({ err });
        resolve({ fields, files });
      });
    });

    const fields = data.fields as IFields;
    const files = data.files as any;
    let fileName = fields.name[0] as string;
    let file = getDataFile(files.file[0]);
    const res = await uploadImage(fileName, file);
    return response.status(200).send(res);
  } catch (error) {
    console.log(error);
    return response.status(500).send({ 
      IpfsHash: null,
      error: "Internal Server Error" 
    });
  }
}
```

That was my Fourth-of-July weekend, and my saturday night. Good night, and God bless America!

I look like this now:
![urbanamerica](/assets/posts/pinata-next/_urbanamerica-midjourney.jpeg)
Photo by <a href="https://www.midjourney.com/jobs/19058b97-7c28-4000-a0d4-7fbea2cd2d49?index=0">urbanamerica from Midjourney</a>