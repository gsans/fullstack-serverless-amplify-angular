# Building your first Fullstack Serverless App with Angular and AWS Amplify

In this workshop we'll learn how to build cloud-enabled web applications with Angular & [AWS Amplify](https://aws-amplify.github.io/).

![](https://i.imgur.com/GW1Nk5B.png)

### Topics we'll be covering:

- [Authentication](#adding-authentication)
- [GraphQL API with AWS AppSync](#adding-a-graphql-api)
- [Mocking and Testing](#local-mocking-and-testing)
- [Predictions](#adding-predictions)
- [Hosting](#hosting)
- [Multiple Environments](#working-with-multiple-environments)
- [Deploying via the Amplify Console](#deploying-via-the-amplify-console)
- [Run locally via Amplify CLI](#run-locally-with-the-amplify-cli)
- [Removing / Deleting Services](#removing-services)

## Pre-requisites

- Node: `13.2.0`. Visit [Node](https://nodejs.org/en/download/current/)
- npm: `6.13.1`. Packaged with Node otherwise run upgrade

```bash
npm install -g npm
```

## Getting Started - Creating the Application

To get started, we first need to create a new Angular project & change into the new directory using the [Angular CLI](https://cli.angular.io/).

If you already have it installed, skip to the next step. If not, either install the CLI & create the app or create a new app using:

```bash
npm install -g @angular/cli
ng new amplify-app
```
 
Now change into the new app directory and make sure it runs

```bash
cd amplify-app
npm install
ng serve
```
## Changes to Angular CLI project

Add type definitions for __Node__ by changing `tsconfig.app.json`. This is a requirement from `aws-js-sdk`.

```json
{
  "compilerOptions": {
    "types": ["node"]
  },
}
```

Add the following code, to the top of `src/polyfills.ts`. This is a requirement for Angular 6+.

```js
(window as any).global = window;

(window as any).process = {
  env: { DEBUG: undefined }
};
```

## Installing the CLI & Initializing a new AWS Amplify Project

Let's now install the AWS Amplify Authentication API & AWS Amplify Angular library:

```bash
npm install --save @aws-amplify/auth @aws-amplify/ui-angular
```
> If you have issues related to EACCESS try using sudo: `sudo npm <command>`.

### Installing the AWS Amplify CLI

Next, we'll install the AWS Amplify CLI:

```bash
npm install -g @aws-amplify/cli
```
> If your installation fails. Try `npm install -g @aws-amplify/cli --unsafe-perm=true`.
> If you have issues related to fsevents with npm install. Try: `npm audit fix --force`.

Now we need to configure the CLI with our credentials:

```js
amplify configure
```

> If you'd like to see a video walkthrough of this configuration process, click [here](https://www.youtube.com/watch?v=fWbM5DLh25U).

Here we'll walk through the `amplify configure` setup. Once you've signed in to the AWS console, continue:
- Specify the AWS Region: __ap-northeast-1 (Tokyo)__
- Specify the username of the new IAM user: __amplify-app__
> In the AWS Console, click __Next: Permissions__, __Next: Tags__, __Next: Review__, & __Create User__ to create the new IAM user. Then, return to the command line & press Enter.
- Enter the access key of the newly created user:   
  accessKeyId: __(<YOUR_ACCESS_KEY_ID>)__   
  secretAccessKey:  __(<YOUR_SECRET_ACCESS_KEY>)__
- Profile Name: __default__

> To view the new created IAM User go to the dashboard at [https://console.aws.amazon.com/iam/home#/users/](https://console.aws.amazon.com/iam/home#/users/). Also be sure that your region matches your selection.

### Initializing A New Project

```bash
amplify init
```

- Enter a name for the project: __amplify-app__
- Enter a name for the environment: __dev__
- Choose your default editor: __Visual Studio Code__   
- Please choose the type of app that you're building __javascript__   
- What javascript framework are you using __angular__   
- Source Directory Path: __src__   
- Distribution Directory Path: __dist/amplify-app__   
- Build Command: __npm run-script build__   
- Start Command: __ng serve__
- Please choose the profile you want to use: __default__
- Do you want to use an AWS profile? __Yes__
- Please choose the profile you want to use __default__

Now, the AWS Amplify CLI has iniatilized a new project & you will see a new folder: __amplify__. The files in this folder hold your project configuration.

```bash
<amplify-app>
    |_ amplify
      |_ .config
      |_ #current-cloud-backend
      |_ backend
      team-provider-info.json
```

## Adding Authentication

To add authentication to our Amplify project, we can use the following command:

```sh
amplify add auth
```

> When prompted choose 
- Do you want to use default authentication and security configuration?: __Default configuration__
- How do you want users to be able to sign in when using your Cognito User Pool?: __Username__
- Do you want to configure advanced settings? __Yes, I want to make some additional changes.__
- What attributes are required for signing up? (Press &lt;space&gt; to select, &lt;a&gt; to 
toggle all, &lt;i&gt; to invert selection): __Email__
- Do you want to enable any of the following capabilities? (Press &lt;space&gt; to select, &lt;a&gt; to toggle all, &lt;i&gt; to invert selection): __None__

> To select none just press `Enter` in the last option.

Now, we'll run the push command and the cloud resources will be created in our AWS account.

```bash
amplify push

Current Environment: dev

| Category | Resource name      | Operation | Provider plugin   |
| -------- | ------------------ | --------- | ----------------- |
| Auth     | amplifyappuuid     | Create    | awscloudformation |
? Are you sure you want to continue? Yes
```


To quickly check your newly created __Cognito User Pool__ you can run

```bash
amplify status
```

> To access the __AWS Cognito Console__ at any time, go to the dashboard at [https://console.aws.amazon.com/cognito/](https://console.aws.amazon.com/cognito/). Also be sure that your region is set correctly.

### Configuring the Angular Application

Now, our resources are created & we can start using them!

The first thing we need to do is to configure our Angular application to be aware of our new AWS Amplify project. We can do this by referencing the auto-generated `aws-exports.js` file that is now in our `src` folder.

To configure the app, open __main.ts__ and add the following code below the last import:

```js
import Auth from '@aws-amplify/auth';
import amplify from './aws-exports';
Auth.configure(amplify);
```

Now, our app is ready to start using our AWS services.

### Importing the Angular Module

Add the Amplify Module and Service to `src/app/app.module.ts`:

```js
import { AmplifyUIAngularModule } from '@aws-amplify/ui-angular';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AmplifyUIAngularModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### Using the Authenticator Component

AWS Amplify provides UI components that you can use in your App. Let's add these components to the project

```bash
npm i --save @aws-amplify/ui
```

Also include these imports to the top of `styles.css`

```css
@import "~@aws-amplify/ui/src/Theme.css";
@import "~@aws-amplify/ui/src/Angular.css";
```

In order to use the Authenticator Component replace all content in __src/app.component.html__ with:

```html
<amplify-authenticator></amplify-authenticator>
```

Now, we can run the app and see that an Authentication flow has been added in front of our App component. This flow gives users the ability to sign up & sign in.

> To view any users that were created, go back to the __Cognito__ dashboard at [https://console.aws.amazon.com/cognito/](https://console.aws.amazon.com/cognito/). Also be sure that your region is set correctly.

Alternatively we can also use

```bash
amplify console auth
```

### Accessing User Data

We can access the user's info now that they are signed in by calling `currentAuthenticatedUser()` which returns a Promise.

```js
import { Component } from '@angular/core';
import Auth from '@aws-amplify/auth';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  constructor() {
    Auth.currentAuthenticatedUser().then(console.log)
  }
}
```

### Custom authentication strategies

The `Authenticator` Component is a really easy way to get up and running with authentication, but in a real-world application we probably want more control over how our form looks & functions.

Let's look at how we might create our own authentication flow.

To get started, we would probably want to create input fields that would hold user input data in the state. For instance when signing up a new user, we would probably need 2 inputs to capture the user's email & password.

To do this, we could create a form like:

```html
<form [formGroup]="signup" (ngSubmit)="onSignup(signup.value)">
  <div>
    <label>Email: </label>
    <input type="email" formControlName="email">
  </div>
  <div>
    <label>Password: </label>
    <input type="password" formControlName="password">
  </div>
  <button type="submit">Submit</button>
</form>
```

We'd also need to have a method that signed up & signed in users. We can us the Auth class to do this. The Auth class has over 30 methods including things like `signUp`, `signIn`, `confirmSignUp`, `confirmSignIn`, & `forgotPassword`. These functions return a promise so they need to be handled asynchronously.

```js
import Auth from '@aws-amplify/auth';

export class SignupComponent implements OnInit {
  public signup: FormGroup;

  constructor(
    private fb: FormBuilder,
  ) { }

  ngOnInit() {
    this.signup = this.fb.group({
      'email': ['', Validators.required],
      'password': ['', Validators.required]
    });
  }

  onSignup(value: any) {
    const email = value.email, password = value.password;
    Auth.signUp(email, password).then( _ => {
      this.success = true;
    }).catch(console.log);
  }
```

## Adding a GraphQL API

To add a GraphQL API, we can use the following command:

```sh
amplify add api
```

Answer the following questions

- Please select from one of the below mentioned services __GraphQL__
- Provide API name: __RestaurantAPI__
- Choose the default authorization type for the API __API key__
- Enter a description for the API key: __(empty)__
- After how many days from now the API key should expire (1-365): __180__
- Do you want to configure advanced settings for the GraphQL API __Yes, I want to make some additional changes.__
- Choose the additional authorization types you want to configure for the API (Press &lt;space&gt; to select, &lt;a&gt; to 
toggle all, &lt;i&gt; to invert selection) __None__
- Do you have an annotated GraphQL schema? __No__
- Do you want a guided schema creation? __Yes__
- What best describes your project: __Single object with fields (e.g., “Todo” with ID, name, description)__
- Do you want to edit the schema now? __Yes__

> To select none just press `Enter`.


> When prompted, update the schema to the following:   

```graphql
type Restaurant @model {
  id: ID!
  clientId: String
  name: String!
  description: String!
  city: String!
}
```

> Note: Don't forget to save the changes to the schema file!

> Next, let's push the configuration to our account:

```bash
amplify push
```

- Are you sure you want to continue? __Yes__
- Do you want to generate code for your newly created GraphQL API __Yes__
- Choose the code generation language target __angular__
- Enter the file name pattern of graphql queries, mutations and subscriptions __src/graphql/**/*.graphql__
- Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions __Yes__
- Enter maximum statement depth [increase from default if your schema is deeply nested] __2__
- Enter the file name for the generated code __src/app/API.service.ts__

Notice your __GraphQL endpoint__ and __API KEY__.

This step created a new AWS AppSync API. Use the command below to access the AWS AppSync dashboard. Make sure that your region is correct.

```bash
amplify console api
```

- Please select from one of the below mentioned services __GraphQL__

### Local mocking and testing

To mock and test the API locally, you can run the `mock` command:

```sh
amplify mock api
```
> Note: local mocking requires Java SDK

- Choose the code generation language target: __javascript__
- Enter the file name pattern of graphql queries, mutations and subscriptions: __src/graphql/**/*.js__
- Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions: __Y__
- Enter maximum statement depth [increase from default if your schema is deeply nested]: __2__

This should open up the local GraphiQL editor.

From here, we can now test the API locally.

### Adding mutations from within the AWS AppSync Console

In the AWS AppSync console, on the left side click on Queries.

Execute the following mutation to create a new restaurant in the API:

```graphql
mutation createRestaurant {
  createRestaurant(input: {
    name: "Nobu"
    description: "Great Sushi"
    city: "New York"
  }) {
    id name description city
  }
}
```

Now, let's query for the restaurant:

```graphql
query listRestaurants {
  listRestaurants {
    items {
      id
      name
      description
      city
    }
  }
}
```

We can even add search / filter capabilities when querying:

```graphql
query searchRestaurants {
  listRestaurants(filter: {
    city: {
      contains: "New York"
    }
  }) {
    items {
      id
      name
      description
      city
    }
  }
}
```

### Interacting with the GraphQL API from our client application - Querying for data

Now that the GraphQL API is created we can begin interacting with it!

The first thing we'll do is perform a query to fetch data from our API. 

To do so, we need to define the query, execute the query, store the data in our state, then list the items in our UI.

> Read more about the __Amplify GraphQL Client__ [here](https://aws-amplify.github.io/docs/js/api#amplify-graphql-client).

First, we will install the AWS Amplify API and PubSub libraries:

```bash
npm install --save @aws-amplify/api @aws-amplify/pubsub
```

To configure the app, open __main.ts__ and change the initial code to configure the new dependencies:

```js
import Auth from '@aws-amplify/auth';
import API from '@aws-amplify/api';
import PubSub from '@aws-amplify/pubsub';
import amplify from './aws-exports';
Auth.configure(amplify);
API.configure(amplify);
PubSub.configure(amplify);
```

```js
import { APIService } from '../API.service';
import { Restaurant } from './types/restaurant';

@Component({
  template: `
    <div>
      <div *ngFor="let restaurant of restaurants">
        {{ restaurant.name }}
      </div>
    </div>`
})
export class AppComponent implements OnInit {
  restaurants: Array<Restaurant>;
  constructor(public api: APIService) { }
  ngOnInit() {
    this.api.ListRestaurants().then(data => {
      this.restaurants = data.items;
    });
  }
}
```

## Performing mutations

 Now, let's look at how we can create mutations.

```js
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { createRestaurant } from '../../graphql/mutations'

@Component(...)
export class HomeComponent implements OnInit {
  public createForm: FormGroup;

  constructor(private fb: FormBuilder) { }

  async ngOnInit() {
    this.createForm = this.fb.group({
      'name': ['', Validators.required],
      'description': ['', Validators.required],
      'city': ['', Validators.required]
    });
    this.api.ListRestaurants().then(event => {
      this.restaurants = event.items;
    });
  } 
  
  public onCreate(restaurant: any) {
    this.api.CreateRestaurant(restaurant).then(event => {
      console.log('item created!');
      this.createForm.reset();
    })
    .catch(e => {
      console.log('error creating restaurant...', e);
    });
  }
}
```

### GraphQL Subscriptions

Next, let's see how we can create a subscription to subscribe to changes of data in our API.

To do so, we need to listen to the subscription, & update the state whenever a new piece of data comes in through the subscription.

```js
@Component(...)
export class HomeComponent implements OnInit {
  ngOnInit() {
    this.api.OnCreateRestaurantListener.subscribe(event => {
      const newRestaurant = event.value.data.onCreateRestaurant;
      this.restaurants = [newRestaurant, ...this.restaurants];
    });
```

## Adding Predictions

To add the predictions category to our Amplify project, we can use the following command:

```sh
amplify add predictions
```

#### Identify Text, Labels and Entities
- Please select from one of the categories below __Identify__
- What would you like to identify? __Identify Text__
- Provide a friendly name for your resource __identifyTextId__
- Would you also like to identify documents? __No__
- Who should have access? __Auth and Guest users__

#### Translate Text
- Please select from one of the categories below __Convert__
- What would you like to convert? __Translate text into a different language__
- Provide a friendly name for your resource __translateTextId__
- What is the source language? __English__
- What is the target language? __Spanish__
- Who should have access? __Auth and Guest users__

#### Generate Speech
- Please select from one of the categories below __Convert__
- What would you like to convert? __Generate speech audio from text__
- Provide a friendly name for your resource __speechGeneratorId__
- What is the source language? __British English__
- Select a speaker __Brian - Male__
- Who should have access? __Auth and Guest users__

#### Transcribe Audio
- Please select from one of the categories below __Convert__
- What would you like to convert? __Transcribe text from audio__
- Provide a friendly name for your resource __transcriptiondId__
- What is the source language? __British English__
- Who should have access? __Auth and Guest users__

#### Interpret Text
- Please select from one of the categories below __Interpret__
- What would you like to interpret? __Interpret Text__
- Provide a friendly name for your resource __interpretTextId__
- What kind of interpretation would you like? __All__
- Who should have access? __Auth and Guest users__


Now, we'll run the push command and the cloud resources will be created in our AWS account.

```bash
amplify push
```

### Configuring the Vue Application

Now, our resources are created & we can start using them!

To configure the app, open __main.js__ and add the following code below the last import:

```js
import Predictions, { AmazonAIPredictionsProvider } from '@aws-amplify/predictions';
Predictions.addPluggable(new AmazonAIPredictionsProvider());
```

Now, our app is ready to start using **Predictions**.

#### Translate Example

The result of `Predictions.convert(config)` will be a promise. If successful will return the translation otherwise will return the input as-is.

```
Predictions.convert({
  translateText: {
    source: {
      text: "My taylor is rich!",
      language: "en"
    },
    targetLanguage: "es"
  }
})
.then(({text}) => {
  this.translation = text;
})
```
> List of [supported languages](https://docs.aws.amazon.com/translate/latest/dg/how-it-works.html#how-it-works-language-codes) to translate from.

#### Detect Language Example

The result of `Predictions.interpret(config)` call will return a promise. If successful, we will be able to pick the main language as part of the results as demonstrated in the following code snipet.

```
Predictions.interpret({
  text: {
    source: {
      text: textToInterpret,
    },
    type: InterpretTextCategories.LANGUAGE
  }
})
.then((result) => {
  let detected = result.textInterpretation.language;
})
```

#### Text to Speech Example

The result of `Predictions.convert(config)` call will return a promise with the resulting audio file which we can reference to play the voice using *HMTL Audio*.

```
Predictions.convert({
  textToSpeech: {
    source: {
      text: textToTranslate,
    },
    voiceId: "Brian"
  }
})
.then((result) => {
  if (result.speech.url) {
    this.audio = new Audio();
    this.audio.src = result.speech.url;
    this.audio.play();
  }
})
```

## Hosting

To deploy & host your app on AWS, we can use the `hosting` category.

```sh
amplify add hosting
```

- Select the environment setup: __DEV (S3 only with HTTP)__
- hosting bucket name __YOURBUCKETNAME__
- index doc for the website __index.html__
- error doc for the website __index.html__

Now, everything is set up & we can publish it:

```sh
amplify publish
```

## Working with multiple environments

You can create multiple environments for your application in which to create & test out new features without affecting the main environment which you are working on.

When you create a new environment from an existing environment, you are given a copy of the entire backend application stack from the original project. When you make changes in the new environment, you are then able to test these new changes in the new environment & merge only the changes that have been made since the new environment was created back into the original environment.

Let's take a look at how to create a new environment. In this new environment, we'll re-configure the GraphQL Schema to have another field for the pet owner.

First, we'll initialize a new environment using `amplify init`:

```sh
amplify init
```

- Do you want to use an existing environment? __N__
- Enter a name for the environment: __apiupdate__
- Do you want to use an AWS profile? __Y__
- __amplify-workshop-user__

Once the new environment is initialized, we should be able to see some information about our environment setup by running:

```sh
amplify env list

| Environments |
| ------------ |
| dev          |
| *apiupdate   |
```

Now we can update the GraphQL Schema in `amplify/backend/api/RestaurantAPI/schema.graphql` to the following (adding the owner field):

```graphql
type Restaurant @model {
  ...
  owner: String
}
```

Now, we can create this new stack by running `amplify push`:

```sh
amplify push
```

After we test it out, we can now merge it into our original dev environment:

```sh
amplify env checkout dev

amplify status

amplify push
```

- Do you want to update code for your updated GraphQL API? __Y__
- Do you want to generate GraphQL statements? __Y__


## Deploying via the Amplify Console

We have looked at deploying via the Amplify CLI hosting category, but what about if we wanted continous deployment? For this, we can use the [Amplify Console](https://aws.amazon.com/amplify/console/) to deploy the application.

The first thing we need to do is [create a new GitHub repo](https://github.com/new) for this project. Once we've created the repo, we'll copy the URL for the project to the clipboard & initialize git in our local project:

```sh
git init

git remote add origin git@github.com:username/project-name.git

git add .

git commit -m 'initial commit'

git push origin master
```

Next we'll visit the Amplify Console in our AWS account at [https://ap-northeast-1.console.aws.amazon.com/amplify/home](https://ap-northeast-1.console.aws.amazon.com/amplify/home).

Here, we'll click __Get Started__ to create a new deployment. Next, authorize Github as the repository service.

Next, we'll choose the new repository & branch for the project we just created & click __Next__.

In the next screen, we'll create a new role & use this role to allow the Amplify Console to deploy these resources & click __Next__.

Finally, we can click __Save and Deploy__ to deploy our application!

Now, we can push updates to Master to update our application.

## Run locally with the Amplify CLI

1. Install and configure the Amplify CLI

```
  npm install -g @aws-amplify/cli
  amplify configure
```

2. Install and configure the Amplify CLI

```
  amplify init --app https://github.com/gsans/ng-china-workshop-solution
```
  
>The init command clones the GitHub repo, initializes the CLI, creates a ‘sampledev’ environment in CLI, detects and adds categories, provisions the backend, pushes the changes to the cloud, and starts the app.

3. Provisioning the frontend and backend

Once the process is complete, the CLI will automatically open the app in your default browser.

## Removing Services

If at any time, or at the end of this workshop, you would like to delete a service from your project & your account, you can do this by running the `amplify remove` command:

```sh
amplify remove auth
amplify push
```

If you are unsure of what services you have enabled at any time, you can run the `amplify status` command:

```sh
amplify status
```

`amplify status` will give you the list of resources that are currently enabled in your app.


## Appendix

### Setup your AWS Account

In order to follow this workshop you need to create and activate an Amazon Web Services account. 

Follow the steps [here](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account)

### Trobleshooting

Message: The AWS Access Key Id needs a subscription for the service

Solution: Make sure you are subscribed to the free plan. [Subscribe](https://portal.aws.amazon.com/billing/signup?type=resubscribe#/resubscribed)


Message: TypeError: fsevents is not a constructor

Solution: `npm audit fix --force`
