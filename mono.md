## Set up continuous deployment and hosting for a monorepo with an Amplify backend 

Amplify Console recently launched better monorepo support, providing developers with mono-repositories a better experience connecting apps to the Amplify Console. A mono-repository is a repository that contains more than one logical project, each in it’s own repository. For example, if you have multiple teams building microsites under your different subdomains of your primary domain, a monorepo strategy gives each team the flexibility to pick their frontend tech stack (e.g. angular vs react), while allowing shared functionality to be stored in common libraries in the same repository. 

With today’s launch Amplify Console makes it easy to deploy monorepo apps with three important features:

1. Automatic detection of build settings when connecting a sub-folder in your mono-repository. This makes connecting and deploying a project in your monorepo frictionless.
2. New builds in the Amplify Console are only triggered when there are code changes within a specific app project.
3. Ability to define the build settings for multiple apps in a single build specification file (amplify.yml) 

In this blog post we are going to walkthrough deploying a React and Angular Todo app that share the same backend built with Amplify.

### Step 1: Set up your monorepo project

To get started, fork the following project.
```
git clone git@github.com:swaminator/monorepo-amplify.git

```
This project contains the frontend code for an angular and react Todo app. The repository has the following structure:
```
> monorepo-amplify
  > react
  > angular
```

### Step 2: Set up the Amplify backend

To set up a backend on AWS, we are going to use the Amplify CLI. The Amplify CLI is a command-line toolchain that simplifies provisioning AWS services. 

```
npm install -g @aws-amplify/cli
```

First, [configure the CLI](https://docs.amplify.aws/cli/start/install#configure-the-amplify-cli) on your machine. Once configured, initialize a new backend project at the root of one of your frontend projects. While we could also initialize the project at the root level, the Amplify is best used attached to one of your projects. This allows you to set up continuous deployment pipelines of the frontend and backend together.

```
> cd monorepo-amplify/react
amplify init
? Enter a name for the project todoreact
? Enter a name for the environment dev
? Choose your default editor: Visual Studio Code
? Choose the type of app that you're building javascript
Please tell us about your project
? What javascript framework are you using react
? Source Directory Path:  src
? Distribution Directory Path: build
? Build Command:  npm run-script build
? Start Command: npm run-script start
Using default provider  awscloudformation

For more information on AWS Profiles, see:
https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html

? Do you want to use an AWS profile? Yes
? Please choose the profile you want to use #enter the profile you created
```

Add api and database
```
> amplify add api
? Please select from one of the below mentioned services: GraphQL
? Provide API name: todo
? Choose the default authorization type for the API API key
? Enter a description for the API key: 
? After how many days from now the API key should expire (1-365): 7
? Do you want to configure advanced settings for the GraphQL API No, I am done.
? Do you have an annotated GraphQL schema? No
? Do you want a guided schema creation? (Y/n) Y
? What best describes your project: Single object with fields (e.g., “Todo” with ID, name, description)
? Do you want to edit the schema now? (Y/n) Y
```

Transform
```
type Todo @model {
  id: ID!
  name: String!
  description: String
}
```

Deploy to cloud
```
amplify push
✔ Successfully pulled backend environment dev from the cloud.

Current Environment: dev

| Category | Resource name | Operation | Provider plugin   |
| -------- | ------------- | --------- | ----------------- |
| Api      | todo          | Create    | awscloudformation |
? Are you sure you want to continue? Yes

The following types do not have '@auth' enabled. Consider using @auth with @model
	 - Todo
Learn more about @auth here: https://docs.amplify.aws/cli/graphql-transformer/directives#auth


GraphQL schema compiled successfully.

Edit your schema at /Users/nsswamin/workspace/Experiments/monorepo-amplify/react/amplify/backend/api/todo/schema.graphql or place .graphql files in a directory at /Users/nsswamin/workspace/Experiments/monorepo-amplify/react/amplify/backend/api/todo/schema
? Do you want to generate code for your newly created GraphQL API Yes
? Choose the code generation language target javascript
? Enter the file name pattern of graphql queries, mutations and subscriptions src/graphql/**/*.js
? Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions Yes
? Enter maximum statement depth [increase from default if your schema is deeply nested] 2
```

Test the React app locally
