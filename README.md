# Burn Book tutorial

Let's create a digital burn book that actually puts privacy first.

🚀 **Your mission**

Build [a Web5 Burn Book](https://burn-book-mean-girls.vercel.app/), based on Mean Girls, but this one is actual private.

🛠️ **Your toolkit**

To build this app, you will wield the following technologies:

- [React](https://react.dev/) - A JavaScript library, your brush to paint user interfaces, craft reusable components, and manage the state of your creation.
- [Next.js](https://nextjs.org/) - A React metaframework, your compass to guide you through creating pages, managing routes, and managing server-side rendering.
- [Web5.js](https://developer.tbd.website/api/web5-js/) - A Web5 JavaScript SDK, your key to creating decentralized applications, facilitating direct communication between users, and granting them sovereignty over their data and identity.

## Level 1: Explore the starter code

Good news! You don't have to start from scratch. There is a pre-built application that contains the user interface for our Burn Book. Right now, the functions in the code don’t do much. They just print messages to the console and don’t have any real actions. As you progress through the tutorial, you will add logic to the functions.
### 📝 Task

To access the starter code, follow the instructions below or explore the project directly in CodeSandbox.

```bash
git clone https://github.com/galaxy-bytes/starter-burn-book-mean-girls
cd starter-burn-book-mean-girls
npm install
npm run dev
```

[![Edit galaxy-bytes/starter-burn-book-mean-girls/main](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/p/github/galaxy-bytes/starter-burn-book-mean-girls/main?import=true&embed=1&file=%2FREADME.md)



### 🕵️ What to Explore
- Navigate to the `src/pages/index.js` file.
- Identify the functions that currently only print console.logs.
- Try to understand how each function interacts.

## Level 2: Connect to Web5 and create a DID
### 📝 Task
Locate the line:
```javascript title="src/pages/index.js"
console.log(`this log is in initWeb5`);
```
and replace it with the following code snippet:
```javascript title="src/pages/index.js"
const { web5, did } = await Web5.connect({sync: '5s'});
setWeb5(web5);
setMyDid(did);
```

### 🧩 Breaking it down
- **Web5.connect()** returns a Web5 instance: It initializes and returns an instance of Web5, allowing you to interact with the Web5 ecosystem.
- **Web5.connect()** also creates or connects to a DID: A DID ([Decentralized Identifier](https://developer.tbd.website/docs/web5/learn/decentralized-identifiers)) is a user’s unique ID card for the decentralized, digital world. It belongs only to you. `Web5.connect()` will either connect to an existing DID that you have or create a new one for you if you don't have one yet. A DID acts as an authenticator for Web5 apps, removing the need for login credentials.

### 📘 Learn more
Learn more about [DID authentication](https://developer.tbd.website/blog/did-authentication).

## Level 3: Define our protocol
### 📝 Task
Locate the line:
```javascript
console.log('this is where we define our protocol')
```

And replace it with the following code snippet:
```javascript
  return {
      protocol: "https://blackgirlbytes.dev/burn-book-finale",
      published: true,
      types: {
        secretMessage: {
          schema: "https://example.com/secretMessageSchema",
          dataFormats: ["application/json"],
        },
        directMessage: {
          schema: "https://example.com/directMessageSchema",
          dataFormats: ["application/json"],
        },
      },
      structure: {
        secretMessage: {
          $actions: [
            { who: "anyone", can: ["create"] },
            { who: "author", of: "secretMessage", can: ["read"] },
          ],
        },
        directMessage: {
          $actions: [
            { who: "author", of: "directMessage", can: ["read"] },
            { who: "recipient", of: "directMessage", can: ["read"] },
            { who: "anyone", can: ["create"] },
          ],
        },
      },
    };
```
### 🧩 Breaking it down
- **protocolDefinition:** This object defines the protocol, its structure, and it grants permissions outlining who can perform specific actions like reading or writing a ding.

### 📘 Learn more
- Learn more about [protocols](https://developer.tbd.website/docs/web5/learn/protocols/).

## Level 4: Query and install
Locate this line:
```js
    console.log('this is in query local protocol')
```

Replace:
```js
return await web5.dwn.protocols.query({
      message: {
        filter: {
          protocol: "https://blackgirlbytes.dev/burn-book-finale",
        },
      },
    });
```
Locate this line
```js
console.log('this is where Query remote protocol is')
```
Replace:
```js
    return await web5.dwn.protocols.query({
      from: did,
      message: {
        filter: {
          protocol: "https://blackgirlbytes.dev/burn-book-finale",
        },
      },
    });
```

Locate this line:
```js
 console.log('this is where we install local protocol')

```

Replace
```js
    return await web5.dwn.protocols.configure({
      message: {
        definition: protocolDefinition,
      },
    });
```

Locate this line:
```js
  console.log('this is where we install remote protocol')
```
cons
Replace:
```js
    const { protocol } = await web5.dwn.protocols.configure({
      message: {
        definition: protocolDefinition,
      },
    });
    return await protocol.send(did);
```


## Level 5: Configure our protocol

### 📝 Task
Locate the line:
```javascript
 console.log('this is where we configure our protocol')
```

And replace it with the following code snippet:
```javascript
 const protocolDefinition = defineNewProtocol();
    const protocolUrl = protocolDefinition.protocol;

    const { protocols: localProtocols, status: localProtocolStatus } = await queryLocalProtocol(web5, protocolUrl);
    if (localProtocolStatus.code !== 200 || localProtocols.length === 0) {
      const result = await installLocalProtocol(web5, protocolDefinition);
      console.log({ result })
      console.log("Protocol installed locally");
    }

    const { protocols: remoteProtocols, status: remoteProtocolStatus } = await queryRemoteProtocol(web5, did, protocolUrl);
    if (remoteProtocolStatus.code !== 200 || remoteProtocols.length === 0) {
      const result = await installRemoteProtocol(web5, did, protocolDefinition);
      console.log({ result })
      console.log("Protocol installed remotely");
    }
```

## Level 6: Writing the messages
Locate the line:
```js
console.log('this is where we Write the secret message')
```

Replace:
```js
 try {
      const secretMessageProtocol = defineNewProtocol();
      const { record, status } = await web5.dwn.records.write({
        data: messageObj,
        message: {
          protocol: secretMessageProtocol.protocol,
          protocolPath: "secretMessage",
          schema: secretMessageProtocol.types.secretMessage.schema,
          recipient: myDid,
        },
      });

      if (status.code === 200) {
        return { ...messageObj, recordId: record.id };
      }

      console.log('Secret message written to DWN', { record, status });
      return record;
    } catch (error) {
      console.error('Error writing secret message to DWN', error);
    }
```
Locate the line
```js
 console.log('this is where we Write the direct message')
```

Replace:
```js
try {
      const directMessageProtocol = defineNewProtocol();
      const { record, status } = await web5.dwn.records.write({
        data: messageObj,
        message: {
          protocol: directMessageProtocol.protocol,
          protocolPath: "directMessage",
          schema: directMessageProtocol.types.directMessage.schema,
          recipient: messageObj.recipientDid,
        },
      });

      if (status.code === 200) {
        return { ...messageObj, recordId: record.id };
      }


      console.log('Direct message written to DWN', { record, status });
      return record;
    } catch (error) {
      console.error('Error writing direct message to DWN', error);
    }try {
      const directMessageProtocol = defineNewProtocol();
      const { record, status } = await web5.dwn.records.write({
        data: messageObj,
        message: {
          protocol: directMessageProtocol.protocol,
          protocolPath: "directMessage",
          schema: directMessageProtocol.types.directMessage.schema,
          recipient: messageObj.recipientDid,
        },
      });

      if (status.code === 200) {
        return { ...messageObj, recordId: record.id };
      }


      console.log('Direct message written to DWN', { record, status });
      return record;
    } catch (error) {
      console.error('Error writing direct message to DWN', error);
    }
```

## Level 7: Take a look at the handle Submit and construct Direct Message methods

## Level 8: Take a look at the fetch Messages methods

## Level 9: Take a look at the delete message method

## Level 10: Let's try it out!



```

