# Implementing Authentication

We have come a long way in this workshop so far: With our models, we have defined the entities needed to run our business; We have created Repositories and Controllers for our models, creating REST API endpoints for us, we have defined the relationships between our models (and gotten even _more_ REST API endpoints), and then we migrated all of that our database! We have also secured the data in flight by using HTTPS instead of HTTP.

There is _one more step_ that we are going to implement. We are going to implement **Authentication**. Authentication allows us to prevent access to our REST API endpoints unless a user or application has proved that they should have access through some credentials. We will be using a Token Authentication strategy, backed by a JWT. You can read more about authentication here: https://loopback.io/doc/en/lb4/Authentication-overview.html

(If this were a real REST API that we were using in our business, we should also implement **Authorization**, but that is pretty involved and we leave no time for the rest of the Hackathon. To learn more about Authorization and LoopBack, read here: https://loopback.io/doc/en/lb4/RBAC-with-authorization.html)

Implementing authentication in LoopBack is actually fairly easy. The steps below are based heavily on LoopBacks own example for doing authentication: https://loopback.io/doc/en/lb4/Authentication-tutorial.html

The first thing we are going to do is download a few LoopBack packages that will help us with the task. In your terminal, make sure that you are in your application's directory, and run

```
npm install @loopback/authentication-jwt
```

This will download several packages into your application, which we can then immediately use for implementing our authentication strategy.

## Creating a file-based DataSource

Before we can do anything with authentication, we should create an additional DataSource that can store the data related to our users. We can just create a simple "In-memory db" DataSource so that we don't have to back it with a database table, it will just be backed by a simple .txt file.

Run the `lb4` CLI again

```
lb4 datasource
```

Then enter the following options:

```
? Datasource name: User
? Select the connector for User:  In-memory db (supported by StrongLoop)
? window.localStorage key to use for persistence (browser only): user
? Full path to file for persistence (server only): user.txt
   create src/datasources/user.datasource.ts

No change to package.json was detected. No package manager install will be executed.
npm WARN config cache-min This option has been deprecated in favor of `--prefer-offline`.

up to date, audited 636 packages in 1s

81 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
   update src/datasources/index.ts

Datasource User was/were created in src/datasources
```

We now have a DataSource that points at a file, instead of a database, to track information about users. Now we just have to update some our files to get authentication working.

## Updating our files

Next, open `src/application.ts` and add the follwing the the end of the consturctor implementation:

```ts
this.component(AuthenticationComponent);
// Mount jwt component
this.component(JWTAuthenticationComponent);
// Bind datasource
this.dataSource(UserDataSource, UserServiceBindings.DATASOURCE_NAME);
```

Then, at the top, add the imports for these objects that we just used:

```ts
import {UserDataSource} from './datasources';
import {AuthenticationComponent} from '@loopback/authentication';
import {
  JWTAuthenticationComponent,
  UserServiceBindings,
} from '@loopback/authentication-jwt';
```

In the end, my entire file looks like:

```ts
import {BootMixin} from '@loopback/boot';
import {ApplicationConfig} from '@loopback/core';
import {
  RestExplorerBindings,
  RestExplorerComponent,
} from '@loopback/rest-explorer';
import {RepositoryMixin} from '@loopback/repository';
import {RestApplication} from '@loopback/rest';
import {ServiceMixin} from '@loopback/service-proxy';
import path from 'path';
import {MySequence} from './sequence';
import {UserDataSource} from './datasources';
import {AuthenticationComponent} from '@loopback/authentication';
import {
  JWTAuthenticationComponent,
  UserServiceBindings,
} from '@loopback/authentication-jwt';

export {ApplicationConfig};

export class TmpApplication extends BootMixin(
  ServiceMixin(RepositoryMixin(RestApplication)),
) {
  constructor(options: ApplicationConfig = {}) {
    super(options);

    // Set up the custom sequence
    this.sequence(MySequence);

    // Set up default home page
    this.static('/', path.join(__dirname, '../public'));

    // Customize @loopback/rest-explorer configuration here
    this.configure(RestExplorerBindings.COMPONENT).to({
      path: '/explorer',
    });
    this.component(RestExplorerComponent);

    this.projectRoot = __dirname;
    // Customize @loopback/boot Booter Conventions here
    this.bootOptions = {
      controllers: {
        // Customize ControllerBooter Conventions here
        dirs: ['controllers'],
        extensions: ['.controller.js'],
        nested: true,
      },
    };

    this.component(AuthenticationComponent);
    // Mount jwt component
    this.component(JWTAuthenticationComponent);
    // Bind datasource
    this.dataSource(UserDataSource, UserServiceBindings.DATASOURCE_NAME);
  }
}
```

This registers these components with the entire application, and binds our datasource with a "UserService" that we are going to create for doing things like registering new users and logging into our API.

Next, we are going to create an empty Controller that will serve as the service for doing these "UserService" things. Before, when we were creating Controllers, we told it to create a CRUD Controller, but here we are going to use an empty Controller. Run the `lb4` CLI again with the `controller` option, name it "User", and select Empty Controller:

```
lb4 controller
? Controller class name: User
Controller User will be created in src/controllers/user.controller.ts

? What kind of controller would you like to generate? Empty Controller
   create src/controllers/user.controller.ts

No change to package.json was detected. No package manager install will be executed.
   update src/controllers/index.ts

Controller User was/were created in src/controllers
```

Ok, this created an empty Controller for use, and created a file called `src/controllers/user.controller.ts`. Right now that file is incredibly bare:

```ts
// Uncomment these imports to begin using these cool features!

// import {inject} from '@loopback/core';


export class UserController {
  constructor() {}
}
```

We need to add our endpoints to it that allow us to actions around logging in. We could talk about how all of it works, but for now I will just give you the code to paste into that file, and you can read about it on LoopBack's website ;)

In `src/controllers/user.controller.ts`, paste the following:

```ts
// Copyright IBM Corp. and LoopBack contributors 2020. All Rights Reserved.
// Node module: @loopback/example-todo-jwt
// This file is licensed under the MIT License.
// License text available at https://opensource.org/licenses/MIT

import {authenticate, TokenService} from '@loopback/authentication';
import {
  Credentials,
  MyUserService,
  TokenServiceBindings,
  User,
  UserRepository,
  UserServiceBindings,
} from '@loopback/authentication-jwt';
import {inject} from '@loopback/core';
import {model, property, repository} from '@loopback/repository';
import {
  get,
  getModelSchemaRef,
  post,
  requestBody,
  SchemaObject,
} from '@loopback/rest';
import {SecurityBindings, securityId, UserProfile} from '@loopback/security';
import {genSalt, hash} from 'bcryptjs';
import _ from 'lodash';

@model()
export class NewUserRequest extends User {
  @property({
    type: 'string',
    required: true,
  })
  password: string;
}

const CredentialsSchema: SchemaObject = {
  type: 'object',
  required: ['email', 'password'],
  properties: {
    email: {
      type: 'string',
      format: 'email',
    },
    password: {
      type: 'string',
      minLength: 8,
    },
  },
};

export const CredentialsRequestBody = {
  description: 'The input of login function',
  required: true,
  content: {
    'application/json': {schema: CredentialsSchema},
  },
};

export class UserController {
  constructor(
    @inject(TokenServiceBindings.TOKEN_SERVICE)
    public jwtService: TokenService,
    @inject(UserServiceBindings.USER_SERVICE)
    public userService: MyUserService,
    @inject(SecurityBindings.USER, {optional: true})
    public user: UserProfile,
    @repository(UserRepository) protected userRepository: UserRepository,
  ) {}

  @post('/users/login', {
    responses: {
      '200': {
        description: 'Token',
        content: {
          'application/json': {
            schema: {
              type: 'object',
              properties: {
                token: {
                  type: 'string',
                },
              },
            },
          },
        },
      },
    },
  })
  async login(
    @requestBody(CredentialsRequestBody) credentials: Credentials,
  ): Promise<{token: string}> {
    // ensure the user exists, and the password is correct
    const user = await this.userService.verifyCredentials(credentials);
    // convert a User object into a UserProfile object (reduced set of properties)
    const userProfile = this.userService.convertToUserProfile(user);

    // create a JSON Web Token based on the user profile
    const token = await this.jwtService.generateToken(userProfile);
    return {token};
  }

  @authenticate('jwt')
  @get('/whoAmI', {
    responses: {
      '200': {
        description: 'Return current user',
        content: {
          'application/json': {
            schema: {
              type: 'string',
            },
          },
        },
      },
    },
  })
  async whoAmI(
    @inject(SecurityBindings.USER)
    currentUserProfile: UserProfile,
  ): Promise<string> {
    return currentUserProfile[securityId];
  }

  @post('/signup', {
    responses: {
      '200': {
        description: 'User',
        content: {
          'application/json': {
            schema: {
              'x-ts-type': User,
            },
          },
        },
      },
    },
  })
  async signUp(
    @requestBody({
      content: {
        'application/json': {
          schema: getModelSchemaRef(NewUserRequest, {
            title: 'NewUser',
          }),
        },
      },
    })
    newUserRequest: NewUserRequest,
  ): Promise<User> {
    const password = await hash(newUserRequest.password, await genSalt());
    const savedUser = await this.userRepository.create(
      _.omit(newUserRequest, 'password'),
    );

    await this.userRepository.userCredentials(savedUser.id).create({password});

    return savedUser;
  }
}
```

Ok, we now have authentication set up for our endpoints, and a service that allows people to create accounts and log in, getting a token that we can then use to authenticate that we should have access to the endpoints.

Start your application and navigate to the explorer. You should notice two things:

1. At the bottom, you now have a `UserController` with endpoints called `/signup`, `/users/login`, and `/whoAmI`. These are endpoints we can use to access our secured endpoints.
2. _All_ of our endpoints now have a little open lock icon next to them, indicate that they are unsecured.

![Our UserController](assets/l.usercontroller.png)

If you try to call `/customer/count`, you should be able to get a response, same as we could before. We have implemented an authentication strategy, but none of our endpoints are secured yet.

At this point, securing our endpoints is super easy. We just need to go into each our our `src/controller/*.controller.ts` files EXCEPT our `user.controller.ts` file, and add the following:

1. Before each Controller class name, add a `@authenticate` decorator.

```ts
@authenticate('jwt') // <---- Add this!
export class CustomerController {
```

2. Above the Controller name, add the `authenticate` import:

```ts
import {authenticate} from '@loopback/authentication';
```

Now if we restart our application, we can try to access our `/customer/count` endpoint again. When you try to access it, you should get back a 401 status code (meaning "Unauthorized")

![401 Unauthorized endpoint](assets/l.unauthorized.png)

Tada! Every endpoint is secured in each of our controllers where we added the `@authenticate('jwt')` decorator.

The get authorized to our endpoints, we first need to create a user account. In the exploer, open the **`UserController`**, expand the `/signup` endpoint, and then click "Try it out". It will allow you to add all sorts of options, but all you really need is `email` and `password`. Enter a payload like:

```json
{
  "email": "youremail@ibm.com",
  "password": "mypassword123"
}
```

Then hit execute. Tada! You have been registered, and can now login though the `/users/login` endpoint. Open that endpoint in the controller, select "Try it out", then enter your credentials in the payload. After clicking "Execute", scroll down and hopefully you will see a 200 HTTP return code.

You should also see in the response body a JSON payload that holds our token that was generated for us:

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjRiMGMwYWU3LWJiYWEtNGY4NC04ZTNmLWZhODZhMmQzNDY4NiIsImVtYWlsIjoibWlyaXNoQGlibS5jb20iLCJpYXQiOjE2NjUxNjE2NDAsImV4cCI6MTY2NTE4MzI0MH0.GG0Bz7_OxCQEkIj3VJQnfDsFT2eieWeOGFlXNX9i2to"
}
```

We are so close! We have our token, but now we need to send it with all of our calls to our endpoints. Scroll all of the way up, and on the top right of the page there is an "Authorize" button. Click it, paste the value of the token, and log in. All of the "unlocked locks" on the page should turn to closed locks, indicating that you are authenticated.

![Now authorized to endpoints](assets/l.authorized.png)

Now, when you try to use your endpoints, you should be authorized to use them! Instead of getting a 401 response code, you will get a 200, and everything should just work!

When implementing your hackathon "solution", just know that you will have to call your `/users/login` endpoint, then pass your token with each call to your other API endpoints.

---
Next: [Hackathon](m.hackathon-time.md)