# Introduction:

Welcome to the LoopBack Hackathon Workshop! In this workshop we are going to be using the knoweldge we gained in the morning deep-dive presentation to create and secure a REST API that serves up data from our Db2 for i database. We will also create frontend and backend services that call our API endpoints, simulating actual traffic to a real REST API.

To accomplish all of this, we are going to:

1. Connect to our IBM i workshop profiles using SSH
2. Run our commands on IBM i using a Bash shell (an environment the software expects)
3. Create a LoopBack application using the CLI `lb4`
4. Create a datasource that uses `loopback-connector-ibmi` to connect LoopBack to the Db2 for i database located on the same system.

Once we get that far, we are going to create an application that serves up data for our fictional business: a bookstore called *Benelux Books*! If you want, you can create a REST API for any purpose you want, but all of the models I will discuss in this workshop will be based on our fictional bookstore. To get our REST API running, we will:

5. Create models using the LoopBack CLI for all of the tables needed to run our bookstore
6. Migrate those models to tables in the Db2 for i database on our IBM i system
7. Create our controllers and repositories to expose our data as REST API endpoints
8. Secure our endpoints using JSON Web Tokens (JWTs)
9. Call our endpoints using Node.js to do some "backend" server work
10. Create a some other application in the Hackathon portion that calls our REST API endpoints.

After that, we will not only have a fully-operation REST API that exposes IBM i resources, but we will also have the knowledge that allows us to integrate those REST API endpoints with other technologies.

If at any point during this presentation you want to go "off-script" and start doing your own thing (make your own website, use a different front-end framework, call our REST APIs from Python or even RPG) feel free! This workshop is intended to be open-ended a freeform, and it will be interesting to see what other people can do with the tools I've given you!

What are we waiting for? Let's get started!

---
Next: [Connecting with SSH](b.connecting.md)