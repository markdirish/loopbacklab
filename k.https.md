# Enabling HTTPS

When you send your credit card information or password information to a website, you always make sure you are using an `https://` website, right? You can usually identify it by a littl green lock symbol in your browser.

The reason that you do this is because when you use HTTP, the information you send is available for everyone on the network to see! They can easily look at the HTTP requests and responses, and read sensitive information tha you might have sent.

The same is true for our LoopBack REST API. When we have been using the explorer to send and receive data, we are sending our information in plain text that anyone who is listening to the lab can read.

Now, we are probably using a VPN to connect to the system, which will do its own encrpytion, but we should be aware that we should try to enable HTTPS for our LoopBack applications to ensure that the data we send, especially confidential information like our user credentials (see next section) are encrypted.

The LoopBack documentation claims that it is easy to set up HTTPS: https://loopback.io/doc/en/lb4/Enabling-https.html

According to them, in your `src/index.ts`, you just need to add

```ts
// At the top of the file...
import * as fs from 'fs';

// Inside the rest object

const config = {
  rest: {
    /// ...
    /// ...
    // Enable HTTPS
    // replace username with your username
    protocol: 'https',
    key: fs.readFileSync('/home/username/key.pem'),
    cert: fs.readFileSync('/home/username/cert.pem'),
  }
}
```

~~However, I tried to generate my own SSL certificates, add them to the correct location, and tell LoopBack to use HTTPS. It refused, and only used HTTP. I wanted to make sure the rest of the hackathon was working, so I never got back to getting it to work.~~

~~If you are a networking guru, and know how to serve up your own self-signed SSL certificates, please teach me how. I love learning new things from you as well as running these workshops!~~

Koen discovered that you can run these commands to get it running with these commands:

```
openssl genrsa -out key.pem 1024
openssl req -new -key key.pem -out certrequest.csr
openssl x509 -req -in certrequest.csr -signkey key.pem -out cert.pem
```

Run these commands, then place the resulting `.pem` files in your `/home/username` directory.

Also, anyone who can secure their endpoints with HTTPS gets a gold star.

---
Next: [Implementing Authentication](l.authentication.md)