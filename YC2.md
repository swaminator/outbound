## Build your app from idea to MVP Part 2

In this post, you will learn how to build a full stack serverless JAMstack ECommerce store using [Gatsby](https://www.gatsbyjs.org/), [AWS Amplify](https://aws-amplify.github.io/) and [JAMstack ECommerce](https://github.com/jamstack-cms/jamstack-ecommerce).

![JAMstack ECommerce](https://dev-to-uploads.s3.amazonaws.com/i/1jxrwpa1mc3qzglkrgl3.jpg)

While this post focuses on the specific use case of building an ECommerce application, the services and features I will showcase are the building blocks for most all real-world production applications, so I hope you will find it useful.

## Laying the groundwork

For this site, I've chosen to use [Gatsby](gatsbyjs.org) in order to get the benefits of a static site, including better performance, better SEO, and cheaper / easier scalability. Next.js (React) and Nuxt (Vue) are other options that would do the job just as well, but I've gone with Gatsby because of my previous experience with it as well as the robust developer community and documentation available at the time of this writing.

The app we will be building has the following features:

1. Ability to query inventory from an API
2. At build time, create navigation based on inventory categories
3. At build time, create pages for each inventory item and category pages for each nav item along with corresponding views
4. Shopping cart / checkout
5. Admin panel for creating / updating inventory
6. Downloading of images at build time to serve from the public folder vs dynamic fetching

Based on these features, we can assume that the app will have the following requirements from an API / service standpoint:

1. Authentication (sign up, sign in)
2. Dynamic group authorization (only Admin users can view and update inventory)
3. API with create, update, and delete operations
4. Public API access for querying the API
5. Private API access so that only Admin users can create / update / delete inventory
6. Image / asset hosting

To build out these features on both the front and the back end we will be using the Amplify Framework:

- __Amplify CLI__ for creating and configuring AWS services
- __Amplify Client__ libraries for interacting with the services
- __Amplify Console__ to host and view the app and features after they are deployed.

Let’s start building!

__To follow along with this tutorial, you need to have an AWS account (sign up [here](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/))__

## Getting started

To get started, clone the [Gatsby JAMstack ECommerce starter project](https://github.com/jamstack-cms/jamstack-ecommerce) that will serve as the base of the application we'll be building:

```sh
$ git clone https://github.com/jamstack-cms/jamstack-ecommerce.git
```

Next, change into the directory and install the dependencies using __npm__ or __yarn__:

```sh
$ cd jamstack-ecommerce

$ npm install

# or

$ yarn
```

Next, start the project to get an idea of how the app will look:

```sh
$ gatsby develop
```

When the app loads, you should be able to go to [http://localhost:8000/](http://localhost:8000/) and see something like this:
![JAMstack ECommerce](https://dev-to-uploads.s3.amazonaws.com/i/bbfmuhzliip4iamooldb.png)

Great, we're now up and running!

You may be wondering where the inventory is coming from. Starting off, the inventory is hard-coded in the inventory file located at __providers/inventory.js__.

This is not ideal though because keeping up with everything locally is hard to scale. Instead, we propose to make the inventory dynamic and be able to add and update inventory via and admin panel using some type of content management system.

To do so, we'll need to set up an API. To start, create the Amplify project so we can begin migrating the inventory provider to a real back end provider.

### Installing Amplify and initializing an Amplify project

Before you can use Amplify, you'll first need to have or [create an AWS Account](https://aws.amazon.com/resources/create-account/).

Next, install the Amplify CLI globally from the command line:

```sh
$ npm install -g @aws-amplify/cli
```


If the CLI is installed, you should be able to run the amplify command and see some output and help options. 

```sh
$ amplify
```


Now that the CLI is successfully installed, we now need to configure the CLI. To do so, run the configure command:

```sh
$ amplify configure
```

This will walk you through the steps to create and configure AWS user credentials locally. For a guided walkthrough of these configuration steps, check out [this video](https://www.youtube.com/watch?v=n4DuYTzpvdE).

### Creating the Amplify project

After the CLI has been configured you can create a new Amplify project:

```sh
$ amplify init

? Enter a name for the project: jamstack-ecommerce
? Enter a name for the environment dev
? Choose your default editor: <your_preferred_editor>
? Choose the type of app that youre building: javascript
? What javascript framework are you using: react
? Source Directory Path: src
? Distribution Directory Path: public
? Build Command: gatsby build
? Start Command: npm run start
```

- When prompted for an AWS profile, choose the profile you created in the configuration step.


After the initialization has been completed, you should now see 2 artifacts created for you in your project directory:

1. __src/aws-exports.js__ - This file will hold the key value pairs of the resource information for the services created by the CLI.
2. __amplify__ directory - This will hold the back end code we write for things like GraphQL schemas and serverless functions managed by the AWS services we'll be using.

Now that we have the base project set up, let's also go ahead and install the AWS Amplify client library:

```sh
$ npm install aws-amplify

# or

$ yarn add aws-amplify
```

## Creating the back end services

Now we are ready to go and can start creating the services we'll be integrating into the app. Let's first start with authentication.

### Authentication

The authentication setup for this app will need to accomplish the following things:


1. Enable users to sign up and sign in
2. Detect Admin users based on a predetermined list of admins and place them in the Admin group once they sign up.

We can do this with a combination of [Amazon Cognito](https://aws.amazon.com/cognito/) (managed authentication service) and [AWS Lambda](https://aws.amazon.com/lambda/) (functions as a service).

We'll create an authentication service that will call (trigger) a Lambda function when someone signs up (post-confirmation). In that function we can determine whether or not they will be allowed Admin access based on their email address.

To create the service, we'll use the Amplify __add__ command:

```sh
$ amplify add auth

? Do you want to use the default authentication and security configuration? Default configuration
? How do you want users to be able to sign in? Username
? Do you want to configure advanced settings? Yes
? What attributes are required for signing up? Email (keep defaults)
? Do you want to enable any of the following capabilities? Add User to Group
? Enter the name of the group to which users will be added. Admin
? Do you want to edit your add-to-group function now? Y
```

Now, let's edit the code for the post-confirmation Lambda trigger. In __amplify/backend/function/function_name/src/add-to-group.js__, use the following code:

```javascript
// amplify/backend/function/function_name/src/add-to-group.js
const aws = require('aws-sdk');

exports.handler = async (event, context, callback) => {
  const cognitoidentityserviceprovider = new aws.CognitoIdentityServiceProvider({ apiVersion: '2016-04-18' });

  // Here, update the array to include the Admin emails you would like to use
  let adminEmails = ["dabit3@gmail.com"], isAdmin = false

  if (adminEmails.indexOf(event.request.userAttributes.email) !== -1) {
    isAdmin = true
  }

  if (isAdmin) {
    const groupParams = {
      GroupName: process.env.GROUP, UserPoolId: event.userPoolId,
    };
  
    const addUserParams = {
      ...groupParams, Username: event.userName,
    };
  
    try {
      await cognitoidentityserviceprovider.getGroup(groupParams).promise();
    } catch (e) {
      await cognitoidentityserviceprovider.createGroup(groupParams).promise();
    }
  
    try {
      await cognitoidentityserviceprovider.adminAddUserToGroup(addUserParams).promise();
      callback(null, event);
    } catch (e) {
      callback(e);
    }
  } else {
    callback(null, event);
  }
};
```

Update the `adminEmails` array to include the emails you'd like to allow Admin access.

This function will add a user to the Admin group if their email is included in the `adminEmails` array.

### Storage

Next, let's create the image storage service using [Amazon S3](https://aws.amazon.com/s3/):

```sh
$ amplify add storage

? Please select from one of the below mentioned services: Content
? Please provide a friendly name for your resource...: <resource_name>
? Please provide bucket name: <some_unique_bucket_name>
? Who should have access: Auth and guest users
? What kind of access do you want for Authenticated users? create, update, read, delete
? What kind of access do you want for Guest users? read
? Do you want to add a Lambda Trigger for your S3 Bucket? N
```

### API & database

The last thing we need to create is an API and a database to store our data. This API needs to allow both authenticated and unauthenticated access.

Authenticated __Admin__ users should be able to create and update items in the database while unauthenticated access will allow us to query the API at build time to fetch the data needed for the application. 

To allow this, we'll create an [AWS AppSync](https://aws.amazon.com/appsync/) GraphQL API & [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) NoSQL database using the CLI:

```sh
$ amplify add api

? Please select from one of the below mentioned services: GraphQL
? Provide API name: furnitureapi
? Choose the default authorization type for the API: Amazon Cognito User Pool
? Do you want to configure advanced settings for the GraphQL API: Yes
? Configure additional auth types? Y
? Choose the additional authorization types you want to configure for the API: API Key
? Enter a description for the API key: gatsby
? After how many days from now the API key should expire: 100
? Configure conflict detection? N
? Do you have an annotated GraphQL schema? N
? Do you want a guided schema creation? Y
? What best describes your project: Single object with fields
? Do you want to edit the schema now? Y
```

This should open the GraphQL schema located at __amplify/backend/api/postershop/schema.graphql__. Here, update the schema to be the following:

```graphql
type Product @model
  @auth(rules: [
    { allow: public, operations: [read] },
    { allow: groups, groups: ["Admin"] }
  ]) {
  id: ID!
  categories: [String]!
  price: Float!
  name: String!
  image: String!
  description: String!
  currentInventory: Int!
  brand: String
}
```

This GraphQL schema has a few additional directives that you might not see on a traditional schema:

__@model__ - This directive will scaffold out a DynamoDB database, addition CRUD (Create, Read, Update, Delete) & List GraphQL schema operations, and GraphQL resolvers mapping between the operations and the database.

__@auth__ - This directive allows us to set up authorization rules on either a GraphQL type or field.


> These directives are part of the GraphQL Transform library of Amplify. To learn more about this library and these directives, check out the documentation [here](https://aws-amplify.github.io/docs/cli-toolchain/graphql#graphql-transform).


In the schema we've created, we want to have two authorization types:

1. Admin users can perform all operations
2. Public access to read items

The services should now be configured and can be deployed to AWS. To do so, we can run the push command:

```sh
$ amplify push --y
```

All of the services have now been deployed and we can start integrating them into the the client application!

To view the AWS services that have been created at any time, open the Amplify console with the following command:

```sh
$ amplify console
```

## Client integration

Now that the back end services are deployed, the next thing we need to do is configure the Gatsby project to recognize the Amplify project. To do so, open __gatsby-browser.js__ and add the following code:

```javascript
import Amplify from 'aws-amplify'
import config from './src/aws-exports'
Amplify.configure(config)
```

### Client authentication

Once the client app is configured, implement authentication for the admin panel. To do so, open __src/pages/admin.js__ and import the `Auth` class from Amplify:

```javascript
// src/pages/admin.js
import { Auth } from 'aws-amplify'
```

Next, modify the `signUp`, `confirmSignUp`, `signIn`, and `signOut` methods to the following:

```javascript
signUp = async (form) => {
  const { username, email, password } = form
  // step 1: Sign up a new user
  await Auth.signUp({
    username, password, attributes: { email }
  })
  this.setState({ formState: 'confirmSignUp' })
}
confirmSignUp = async (form) => {
  const { username, authcode } = form
  // step 2: Use MFA to confirm the new user
  await Auth.confirmSignUp(username, authcode)
  this.setState({ formState: 'signIn' })
}
signIn = async (form) => {
  const { username, password } = form
  // step 3: Sign in the new user
  await Auth.signIn(username, password)
  // step 4: Check to see if the user is an Admin, if so, show the inventory view.
  const user = await Auth.currentAuthenticatedUser()
  const { signInUserSession: { idToken: { payload }}} = user
  if (payload["cognito:groups"] && payload["cognito:groups"].includes("Admin")) {
    this.setState({ formState: 'signedIn', isAdmin: true })
  }
}
signOut = async() => {
  // allow users to sign out
  await Auth.signOut()
  this.setState({ formState: 'signUp' })
}
```


Authentication is now enabled and users can begin signing up and signing in to view the inventory.

> In the bottom right navigation, click on __Admins__ to view the admin panel to sign up and sign in

In this component, we use a few different methods on the Auth class like `signUp` and `signIn`. Auth has over 30 different methods for handling user authentication. To learn more, check out the documentation [here](https://aws-amplify.github.io/docs/js/authentication) or the API [here](https://aws-amplify.github.io/amplify-js/api/classes/authclass.html).

Next, let's test it out:

```sh
$ gatsby develop
```

You'll notice that when you sign in and refresh the page, the user state is not persisted. We can fix this by checking to see if the user is signed in when the app loads. To do so, update `componentDidMount` with the following code:

```javascript
async componentDidMount() {
  const user = await Auth.currentAuthenticatedUser()
  const { signInUserSession: { idToken: { payload }}} = user
  if (payload["cognito:groups"] && payload["cognito:groups"].includes("Admin")) {
    this.setState({ formState: 'signedIn', isAdmin: true })
  }
}
```

### Client API integration

Now that we have authentication working, let's use the API to create and update data in our app. To do so, we'll be first working with the inventory provider located at __src/templates/ViewInventory.js__. Here, let's update it to fetch data from our real API.

First, import GraphQL query and the APIs needed from AWS Amplify:

```javascript
// src/templates/ViewInventory.js
import { API, graphqlOperation } from 'aws-amplify'
import { listProducts } from '../graphql/queries'
```

Next, update the `fetchInventory` method to fetch the data from the API:

```javascript
fetchInventory = async() => {
  const inventoryData = await API.graphql(graphqlOperation(listProducts))
  const { items } = inventoryData.data.listProducts
  console.log("inventory items: ", items)
  this.setState({ inventory: items })
}
```


You'll notice that when we run the app and __console.log__ the items coming back from the API, there is an empty array. This is because we have yet to create any real items in our database.

To add the ability to create items, we'll need to make some updates to __src/components/formComponents/AddInventory.js__.

First, update the imports to add the following:

```javascript
// src/components/formComponents/AddInventory.js
import { Storage, API, graphqlOperation } from 'aws-amplify'
import { createProduct } from '../../graphql/mutations'
import uuid from 'uuid/v4'
```

Next, update the `onImageChange` and `addItem` methods to the following:

```javascript
onImageChange = async (e) => {
  const file = e.target.files[0];
  const fileName = uuid() + file.name
  // save the image in S3 when it's uploaded
  await Storage.put(fileName, file)
  this.setState({ image: fileName  })
}
addItem = async () => {
  const { name, brand, price, categories, image, description, currentInventory } = this.state
  if (!name || !brand || !price || !categories.length || !description || !currentInventory || !image) return

  // create the item in the database
  const item = { ...this.state, categories: categories.replace(/\s/g, "").split(',') }
  await API.graphql(graphqlOperation(createProduct, { input: item }))
  this.clearForm()
}
```


Now, you'll notice that you can create items and when we view the inventory, they show up!

### Client Storage integration

One odd thing you'll notice is that the images do not show up in the inventory view. This is because we are attempting to render an image key from S3 that is not yet signed. We can fix this by opening the image component at __src/components/image.js__ and adding image signing from S3.

We will check to see if the image is a locally downloaded image (if the image path includes __downloads__). If it does not, then we know it is a remote image from S3 and we will fetch the signed URL for the image.

First, import the `Storage` class from Amplify:

```javascript
// src/components/image.js
import { Storage } from 'aws-amplify'
```

Next, update the `fetchImage` function to this:

```javascript
async function fetchImage(src, updateSrc) {
  if (!src.includes('downloads')) {
    const image = await Storage.get(src)
    updateSrc(image)
  } else { updateSrc(src) }
}
```

Now, we should see the images rendered in the list.

We next need to enable the editing and deleting of items. You'll notice that if you edit an item in the Admin view and refresh,  the changes do not persist. To fix that, open __src/templates/ViewInventory.js__ and make the following changes.

First, import the `updateProduct` and `deleteProduct` mutations:

```javascript
// src/templates/ViewInventory.js
import { updateProduct, deleteProduct } from '../graphql/mutations'
```

Next, update the `saveItem` and `deleteItem` methods to the following:

```javascript
saveItem = async index => {
  const inventory = [...this.state.inventory]
  inventory[index] = this.state.currentItem
  await API.graphql(graphqlOperation(updateProduct, { input: this.state.currentItem }))    
  this.setState({ editingIndex: null, inventory })
}

deleteItem = async index => {
  const id = this.state.inventory[index].id
  const inventory = [...this.state.inventory.slice(0, index), ...this.state.inventory.slice(index + 1)]
  this.setState({ inventory })
  await API.graphql(graphqlOperation(deleteProduct, { input: { id }}))
}
```

Now when we save an item, the updates also go to the database!

### Build-time API integration

Finally, we need to change the build step to use the new API we've created instead of the hard-coded inventory data we've created. When we run __gatsby develop__ or __gatsby build__, we will use the public API access to enable the system to query the data from the API and use it for the app.

We also want to include in the build step a way to download the images locally in our project so we are not fetching remote images, instead we are rendering a local copy of the image that we will be downloading and storing in a local __downloads__ directory in the __public__ folder.

For this to work, first create at least 4 items in your inventory from the __admin__ panel.

Next, create __downloadImage.js__ in the __utils__ folder. This function will allow us to download images locally using the file system (`fs`) module:

```javascript
// utils/downloadImage.js
import fs from 'fs'
import axios from 'axios'
import path from 'path'

function getImageKey(url) {
  const split = url.split('/')
  const key = split[split.length - 1]
  const keyItems = key.split('?')
  const imageKey = keyItems[0]
  return imageKey
}

function getPathName(url, pathName = 'downloads') {
  let reqPath = path.join(__dirname, '..')
  let key = getImageKey(url)
  key = key.replace(/%/g, "")
  const rawPath = `${reqPath}/public/${pathName}/${key}`
  return rawPath
}

async function downloadImage (url) {
  return new Promise(async (resolve, reject) => {
    const path = getPathName(url)
    const writer = fs.createWriteStream(path)
    const response = await axios({
      url,
      method: 'GET',
      responseType: 'stream'
    })
    response.data.pipe(writer)
    writer.on('finish', resolve)
    writer.on('error', reject)
  })
}

export default downloadImage 
```

Now open __gatsby-node.esm.js__. Add the following imports and statements at the top of the file:

```javascript
// gatsby-node.esm.js
import config from './src/aws-exports'
import axios from 'axios'
import tag from 'graphql-tag'
import fs from 'fs'
import downloadImage from './utils/downloadImage'
import Amplify, { Storage } from 'aws-amplify'
Amplify.configure(config)

const graphql = require('graphql')
const { print } = graphql
```

Next, create a new function called `fetchInventory` to fetch inventory from our new API and place the function anywhere in __gatsby-node.esm.js__.

This function will also map over all of the inventory items and download the images locally at build time using the `downloadImage` function that we created in the previous step:

```javascript
async function fetchInventory() {
  /* new */
  const listProductsQuery = tag(`
    query listProducts {
      listProducts(limit: 500) {
        items {
          id
          categories
          price
          name
          image
          description
          currentInventory
          brand
        }
      }
    }
  `)
  const gqlData = await axios({
    url: config.aws_appsync_graphqlEndpoint,
    method: 'post',
    headers: {
      'x-api-key': config.aws_appsync_apiKey
    },
    data: {
      query: print(listProductsQuery)
    }
  })

  let inventory = gqlData.data.data.listProducts.items

  if (!fs.existsSync(`${__dirname}/public/downloads`)){
    fs.mkdirSync(`${__dirname}/public/downloads`);
  }

  await Promise.all(
    inventory.map(async (item, index) => {
      try {
        const relativeUrl = `../downloads/${item.image}`
        if (!fs.existsSync(`${__dirname}/public/downloads/${item.image}`)) {
          const image = await Storage.get(item.image)
          await downloadImage(image)
        }
        inventory[index].image = relativeUrl
      } catch (err) {
        console.log('error downloading image: ', err)
      }
    })
  )
  return inventory
}
```

Finally, in `exports.sourceNodes` and `exports.createPages` update the calls to `getInventory` with the new `fetchInventory` functions:

```javascript
/* replace const inventory = await getInventory() with this 👇 */
const inventory = await fetchInventory()
```

Run the `develop` command to test it out:

```sh
$ gatsby develop
```

Running a new build will fetch the data from the GraphQL API and create a new navigation based on the updated product categories and also build out a new static version of the site.

## Conclusion

At this point, you are up and running with an MVP of a real-world and scalable ECommerce application running on AWS!

From here, you may want to dive deeper on the Amplify documentation to learn more about the APIs and services we’ve used as well as the other APIs that we’ve not yet worked with.

__So far we've set up the following features:__
- [Authentication](https://aws-amplify.github.io/docs/js/authentication) (Amazon Cognito)
- [API](https://aws-amplify.github.io/docs/js/api) (AWS AppSync)
- [Storage] (https://aws-amplify.github.io/docs/js/storage) (Amazon S3)

__You might also be interested in learning about:__
- [Predictions](https://aws-amplify.github.io/docs/js/predictions) (ML / AI)
- [Interactions](https://aws-amplify.github.io/docs/js/interactions) (chat bots, Amazon Lex)
- [Analytics](https://aws-amplify.github.io/docs/js/analytics) (Amazon Pinpoint)
- [REST APIs](https://aws-amplify.github.io/docs/js/api#using-rest) (AWS Lambda + Amazon API Gateway)

### Next steps

Here are a few things you can do to continue improving this app.

__Hosting - deploy to the Amplify Console__

If you host your app in GitHub, BitBucket, GitLab, or AWS CodeCommit, you can easily deploy the entire site to live hosting and add a custom domain in just a few minutes using the [Amplify Console](http://console.aws.amazon.com/amplify). To see a quick video of how to do this with a Gatsby site in less than one minute, check out [this video](https://www.youtube.com/watch?v=cl1v4BkZj4w).

__Configure server-side logic to process the payments with Stripe.__ You can do this easily from where we currently are by adding a serverless function and API using Amplify and the API category:

```sh
$ amplify add function

$ amplify add api

- Choose REST
```

If you’d like to see an example of the function code needed to interact with stripe, check out [this code snippet](https://github.com/jamstack-cms/jamstack-ecommerce/blob/master/snippets/lambda.js).

Also, consider verifying totals by passing in an array of IDs into the function, calculating the total on the server, then comparing the totals to check and make sure they match.

__Update inventory items as they are purchased__

To keep the inventory up to date, you probably want to decrement the inventory as a purchase is made. To do this, you could send an update request to decrement the database before an order was confirmed.

To make this even more secure, you could use a [DynamoDB Transaction](https://docs.aws.amazon.com/appsync/latest/devguide/tutorial-dynamodb-transact.html) to only process the order if there was enough in the inventory and decrement the number of items if there are any in the inventory in a single operation.

To learn more about Amplify, check out these resources:
[Documentation](https://aws-amplify.github.io/)
[Awesome AWS Amplify](https://github.com/dabit3/awesome-aws-amplify)
[My YouTube](https://www.youtube.com/naderdabit)

Follow me on Twitter at [dabit3](https://twitter.com/dabit3)
