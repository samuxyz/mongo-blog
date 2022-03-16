---
title: "Deploy a MERN stack with Express, React and MongoDB Atlas serverless database"
description: "This first part of MERN series shows how to build a Blog application and deploy it on Koyeb Serverless Platform."
tags: ["MongoDB", "Node", "Express", "React", "MERN"]
editUrl: "pages/tutorials/deploy-a-mern-stack-with-express-react-and-mongodb-atlas-serverless-database"
lastEdited: "2022-03-15T00:00:00.000Z"
authors:
  - fullname: "Samuele Zaza"
    avatar: "/static/images/team/szaza.jpg"
---

## Introduction

This is the first article of a mini-series focused on the popular MERN (Mongo, Express, React, Node) stack.
We will create a Blog sample app as it gives us a chance to dig into the MERN stack, building the fundamental blocks and connect the dots. At the end we should have a full-functioning basic Blog web app where authors can post, edit and delete articles. To complete the tutorial, the app will be deployed on the internet by using Koyeb serverless platform.
The second part, instead, will focus on creating a microservice to add extra search capabilities to the blog app by using mongo search atlas.

## Requirements

- A local environment with NPM and Node.js installed
- Basic knowledge of JavaScript, Express, React and querying a MongoDB database using Mongoose
- A [GitHub account](https://github.com) to version and deploy your application code on Koyeb
- A [Koyeb account](https://app.koyeb.com) to deploy and run the application

## Steps

1. Set up the project
2. Create a MongoDB Atlas database
3.
4.

## Set up the project

### Create the project client and server repos

Let's kick off the project by creating the repo `mongo-blog` and installing all the related dependencies.

Open your terminal and create the project folder:
```bash
mkdir mongo-blog
```

Move into `mongo-blog` first and setup Express using `express-generator`:

```bash
cd mongo-blog
npx express-generator
```

You will be prompted several questions to create the `package.json` file such as the project's name, version, and more.
Add the following code to the `package.json` file::

```
{
  "name": "mongo-blog-server",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {
    "cookie-parser": "~1.4.4",
    "debug": "~2.6.9",
    "express": "~4.16.1",
    "http-errors": "~1.6.3",
    "jade": "~1.11.0",
    "morgan": "~1.9.1"
  }
}
```

Next, we are going to add 2 more packages:

- `nodemon` to reload the server. As we are developing in our local environment, we want our server to reload whenever a change in the code occurs.
- `cors` to allow cross-origin resource sharing. This is important when the React-based client calls the server API in our local environment.

In your terminal go ahead and install them:

```bash
yarn add nodemon --save-dev
yarn add cors
```

Open your `package.json` and add one more command under `scripts`:

```bash

{
  "name": "mongo-blog-server",
  "version": "0.0.0",
  "private": true,
  "scripts": {
+   "dev": "nodemon ./bin/www",
    "start": "node ./bin/www"
  },
  "dependencies": {
    "cookie-parser": "~1.4.4",
+   "cors": "^2.8.5",
    "debug": "~2.6.9",
    "express": "~4.16.1",
    "http-errors": "~1.6.3",
    "jade": "~1.11.0",
    "morgan": "~1.9.1"
  },
+ "devDependencies": {
+   "nodemon": "^2.0.15"
+ }
}
```

In `app.js` we are going to require `cors` and attach it to the app:

```javascript
...
const cors = require('cors');
...
app.use(cors());
```

To model our application data and connect to a Mongo database to store posts we are going to use `mongoose` as it is a very straight-forward ORM built for Node:

```bash
yarn add mongoose
```

Finally, we need to add an extra script to build the client `bundle.js` to be returned:

```bash
{
  "name": "mongo-blog",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "dev": "nodemon ./bin/www",
    "start": "node ./bin/www",
+   "build-client": "cd ./client && yarn build"
  },
  "dependencies": {
    "cookie-parser": "~1.4.4",
    "cors": "^2.8.5",
    "debug": "~2.6.9",
    "express": "~4.16.1",
    "http-errors": "~1.6.3",
    "jade": "~1.11.0",
    "mongoose": "^6.2.3",
    "morgan": "~1.9.1"
  },
  "devDependencies": {
    "nodemon": "^2.0.15"
  }
}
```

Let's run `npm install` in the terminal and we can now move to the client setup.

First, create a folder `/client` and install React using `create-react-app`:

```bash
npx create-react-app .
```

Similarly to `express-generator` the command will create a ready-to-go React project hiding most of the tedious configurations required in the past.

On top of the basic packages (react, react-dom...) we have to carefully think about what our blog client needs:

- Clearly, the client should make API calls to the server to perform basic CRUD operations on the database.
- There are gonna be different pages to create, read, edit and delete blog posts.
- We can assume there is need for forms to create and edit a post.

Those are very common functionalities and the npm ecosystem offers tons of different packages. For the purpose of the tutorial, we are gonna install `axios` to make API calls, `react-router-dom` to handle client routing and `react-hook-form` to submit form data.

In the terminal, go ahead and install them:

```bash
yarn add axios react-router-dom react-hook-form
```

As both server and client share the same repo, the folder `/public` in the root will be used to return the static client after building it. To do so, we need to tweak the "build" script inside `/client/package.json` to build the static files in it:

```javascript
{
  "name": "mongo-blog-client",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^5.16.2",
    "@testing-library/react": "^12.1.3",
    "@testing-library/user-event": "^13.5.0",
    "axios": "^0.26.0",
    "bootstrap": "5.1.3",
    "react": "^17.0.2",
    "react-bootstrap": "^2.1.2",
    "react-dom": "^17.0.2",
    "react-hook-form": "^7.27.1",
    "react-router-dom": "^6.2.1",
    "react-scripts": "5.0.0",
    "web-vitals": "^2.1.4"
  },
  "scripts": {
    "start": "react-scripts start",
+   "build": "BUILD_PATH='../public' react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

Lastly, let's talk about styling. We don't really want to spend too much time dealing with CSS so we are using Bootstrap, specifically `react-bootstrap` so that we can include all the UI components we need without really adding CSS:

```bash
yarn add bootstrap@5.1.3 react-bootstrap
```

### Create a database on Mongo Atlas

The easiest way to setup our MongoDB database is to rely on a cloud service such as [Mongo Atlas](https://www.mongodb.com/cloud/atlas/lp/try2). They host databases on AWS, Google Cloud, Azure and provide several options to operate and scale MongoDB painlessly.

After signing-up you will be asked to fill-in a quick survey about the project.

In the plan page, choose the "shared" plan which starts for free:
[mongo-plan]

Next, choose the cloud provider and region, leave the default values for "cluster tier", "additional settings" and choose a cluster name, "mongo-blog-db".

[mongo-deployment-1]
[mongo-deployment-2]



In the dashboard "quickstart" go ahead and create a username and password as we will need that to connecto the database later on. Plus, we can restrict IP access to the database. This is very important measure when going to production with a real app but for the purpose of the tutorial we can simply add `0.0.0.0/0` to allow access from any IP.

[mongo-db-setup-1]
[mongo-db-setup 2]

The database will successfully deploy in a few minutes, in the meantime we can click on the "connect" button, choose "connect your application" and copy the connection string.

[mongo-connection]
A typical connection string should look  like this:

```
mongodb+srv://samuele:<password>@mongo-client-db.r5bv5.mongodb.net/myFirstDatabase?retryWrites=true&w=majority
```

Move back the codebase, open `app.js` to require mongoose, connect it to the database by using the connection string and recover from potential errors:

```javascript
const mongoose = require('mongoose');
const CONNECTION_STRING = `process.env.CONNECTION_STRING` // env variable we can defined in the package.json containing the connection string and pwd

// setup connection to mongo
mongoose.connect(CONNECTION_STRING);
const db = mongoose.connection;
// recover from errors
db.on('error', console.error.bind(console, 'connection error:'));
```

Since we decided to get the connection string as an environment variable, to test it in development we can add it to the `package.json`:

```bash
{
  "name": "mongo-blog",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "dev": "nodemon ./bin/www",
    "start": "node ./bin/www",
    "build-client": "cd ./client && yarn build"
  },
  "dependencies": {
    "cookie-parser": "~1.4.4",
    "cors": "^2.8.5",
    "debug": "~2.6.9",
    "express": "~4.16.1",
    "http-errors": "~1.6.3",
    "jade": "~1.11.0",
    "mongoose": "^6.2.3",
    "morgan": "~1.9.1"
  },
  "devDependencies": {
    "nodemon": "^2.0.15"
  },
+ "nodemonConfig": {
+   "env": {
+     "CONNECTION_STRING": "YOUR_CONNECTION_STRING_AND_PWD"
+   }
+ }
}

```


## Create the Blog post Model

With the database now up and running, it's time to create our first model `Post`. 
### Define the article schema
The basic schema for a blog post is defined by a title, the content of the post, the author, a creation date and optionally tags. The following should help us visualize the schema:

|  Fields  |      Type      |  Required |
|----------|:-------------:|:------:|
| title |  String | X |
| author |    String   |   X |
| content | String |    X |
| tags |    Array  |    |
| createdAt | Date |    X |

### Implement the schema using Mongoose

Mongoose straightforward syntax makes creating models a very simple operation.

In `mongo-blog-server`create a new folder `/models` and a new file `post.js` in it:

```bash
mkdir models
touch /models/post.js
```

And here is the code:
```
// Dependencies
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

// Defines the Post schema
const PostSchema = new Schema({
  title: { type: String, required: true },
  content: { type: String, required: true },
  author: { type: String, required: true },
  tags: { type: [String] },
  createdAt: { type: Date, default: Date.now },    
});

// Sets the createdAt parameter equal to the current time
PostSchema.pre('save', (next) => {
  now = new Date();
  if (!this.createdAt) {
    this.createdAt = now;
  }

  next();
});

// Exports the PostSchema for use elsewhere.
module.exports = mongoose.model('Post', PostSchema);
```

1. Require Mongoose and use the `Schema` class to create `PostSchema`.
2. When creating the object `PostSchema`, we add the fields title, content, author, tags, createdAt.
3. Instruct `PostSchema` to automatically add the creation date right before saving the new post inside the database for us.
4. We export the model to use it within our controllers to perform CRUD operations on the posts.

## Blog API endpoints with Express

Now that we have completed the modelling of our blog posts we can create API endpoints to work with them. As mentioned earlier, our blog app allows users to write, read, edit and delete posts, so we can code a few endpoints to achieve that. Specifically:

1. GET `/api/posts` returns all the posts in descending order, from the latest to the earliest.
2. GET `/api/posts/:id` returns a single blog post given its id.
3. POST `/api/posts` saves a new blog post into the db.
4. PUT `/api/posts/:id` updates a blog post given its id.
5. DELETE `/api/posts/:id` deletes a blog post.

### Create CRUD endpoints using express routes

Thanks to `express-generator` scaffolding we already have the routes folder `/routes` inside `mongo-blog-server`. In the terminal create a new file `posts.js` inside of it:

```bash
touch /routes/posts.js
```

Using express `Router` object we are going to create each endpoint. The first one, GET `/api/posts` retrieves the posts using our newly create Post model function `find()`, sort them using `sort()` and then return the whole list to the client:

```javascript
const express = require('express');
const router = express.Router();
// Require the post model
const Post = require('../models/post');

/* GET posts */
router.get('/', async (req, res, next) => {
  // sort from the latest to the earliest
  const posts = await Post.find().sort({ createdAt: 'desc' });
  return res.status(200).json({
    statusCode: 200,
    message: 'Fetched all posts',
    data: { posts },
  });
});
```

In one single line of code we fetched and sorted the post, that's Mongoose magic!

We can implement GET `/api/posts/:id` similarly but this time we are using `findById` and we are passing the URL parameter `id`:

```javascript
...

/* GET post */
router.get('/:id', async (req, res, next) => {
 // req.params contains the route parameters and the id is one of them
  const post = await Post.findById(req.params.id);
  return res.status(200).json({
    statusCode: 200,
    message: 'Fetched post',
    data: {
      post: post || {},
    },
  });
});
```

Note that if we cannot find any post with that id, we are still returning a positive 200 HTTP status with an empty object as post.

At this point we have functioning endpoints but without posts in the DB we cannot really do much. Let's fix this by creating a POST `/api/posts` endpoint:
In `req.body` we collect the title, author, content and tags coming from the client, then create a new post save it into the DB:

```javascript
/* POST post */
router.post('/', async (req, res, next) => {
  const { title, author, content, tags } = req.body;
  // Create a new post
  const post = new Post({
    title,
    author,
    content,
    tags,
  });
  // Save the post into the DB
  await post.save();
  return res.status(201).json({
    statusCode: 201,
    message: 'Created post',
    data: { post },
  });
});
```

Next, we want to retrieve and update a post. For this action, we can create a PUT `/api/posts/:id` endpoint while Mongoose provides a handy function `findByIdAndUpdate`:

```javascript
/* PUT post */
router.put('/:id', async (req, res, next) => {
  const { title, author, content, tags } = req.body;
  // findByIdAndUpdate accepts the post id as first parameter and the new values as second parameter
  const post = await Post.findByIdAndUpdate(
    req.params.id,
    { title, author, content, tags },
  );
  
  return res.status(200).json({
    statusCode: 200,
    message: 'Updated post',
    data: { post },
  });
});
```

The only action left is the ability of deleting a specific blog post by sending its id. Mongoose once again provides a function `deleteOne` we can use to tell Mongo to delete the post with that id:

```javascript
...
/* DELETE post */
router.delete('/:id', async (req, res, next) => {
  // Mongo stores the id as `_id` by default
  const result = await Post.deleteOne({ _id: req.params.id });
  return res.status(200).json({
    statusCode: 200,
    message: `Deleted ${result.deletedCount} post(s)`,
    data: {},
  });
});

module.exports = router;
```

We completed our new router, we just have to attach it to our server and test it out with POSTMAN. Open `app.js` and under `indexRouter` go ahead and add `postsRouter` as well:

```javascript
...
app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));
app.use(cors());

app.use('/', indexRouter);
// add postsRouter to the server
app.use('/api/posts', postsRouter);
```

### Testing the endpoints using Postman

In absence of a client we can use [POSTMAN](#https://www.postman.com/) to test our API as it is extremely flexible and easy to use:
It allows us to specificy the type of request (GET, POST, PUT, DELETE etc.), the type of payload, if any, and several other option to fine-tune our tests

We currently have an empty database so the very first test can be the creation of a post: Let's specify we want a POST request to `http://localhost:3001/api/posts`. For the body payload, select `raw` and choose `JSON` in the dropdown so that we can use JSON syntax to create it. Here is the result of the call:

[create-blog-post]

To make sure the post was really created, we can make a call to `http://localhost:3001/api/posts` to get the full list of posts as well as `http://localhost:3001/api/posts/:post_id` to fetch the single post:

[get-all-posts]
[get-post]

NB: Since we have just one post, the result of the API calls should be almost the same as GET `/api/posts` returns an array of posts with a single item in it.

How about updating the post? let's change the title and add an extra tag:

[update-post]

If you are unsure whether it was correctly updated, go ahead and call GET `/api/posts/post_id` again:

[get-updated-post]

it's not time to delete it:

[delete-post]

Run GET `/api/posts` again and you should get an empty list of posts as result:

[get-empty-list-of-posts]

## Blog UI with React

The server-side of the application is now complete so it's time work on the client.

### Client routes and basic layout

One of the very first things to define are the routes of our web application: The homepage, singles posts pages, create a new post and edit posts.
Here are the proposed URLs:

|  URL  |      Description  
|----------|:-------------|
| / |  Homepage | X |
| /posts/:post_id | Post content page   
| /posts/new | Page to create a new post 
| /posts/:post_id/edit | Page to edit a post

In our code, the routes will all reside under `/app.js` using `react-router-dom` components `Routes` and `Route`:

```javascript

import { Routes, Route } from 'react-router-dom';
import Home from './pages/home';

function App() {
  return (
      <Routes>
        <Route path="/" element={<Home />} />
      </Routes>
  );
}

export default App;
```

In this example we are rendering the `Home` component when the browser hits the homepage.

`app.js` acts as the root component of our client so we can imagine the shared layout of our blog being render through `App`: The Navbar with a button to create a new post is always visible on every page of our client application so it's only natural to render it here:

```javascript
// Import Bootstrap CSS
import 'bootstrap/dist/css/bootstrap.min.css';
import { Routes, Route } from 'react-router-dom';
import Home from './pages/home';
// Import the Navbar, Nav and Container components from Bootstrap for a nice layout
import Navbar from 'react-bootstrap/Navbar';
import Nav from 'react-bootstrap/Nav';
import Container from 'react-bootstrap/Container';

function App() {
  return (
    <>
      <Navbar bg="dark" expand="lg" variant="dark">
        <Container>
          <Navbar.Brand href="/">My Blog</Navbar.Brand>
          <Navbar.Toggle aria-controls="basic-navbar-nav" />
          <Nav className="me-auto">
            <Nav.Link href="/posts/new">New</Nav.Link>
          </Nav>
        </Container>
      </Navbar>
      <Routes>
        <Route path="/" element={<Home />} />
      </Routes>
    </>
  );
}

export default App;
```

In a few lines of code we created a decent layout that, once we implement `Home`, should look like this:

[react-homepage]

We previously defined the all the client routes, so we can add them all in `App` along with main components we will implement later:

```javascript
import 'bootstrap/dist/css/bootstrap.min.css';
import { Routes, Route } from 'react-router-dom';

// We are going to implement each one of these "pages" in the last section
import Home from './pages/home';
import Post from './pages/post';
import Create from './pages/create';
import Edit from './pages/edit';

import Navbar from 'react-bootstrap/Navbar';
import Nav from 'react-bootstrap/Nav';
import Container from 'react-bootstrap/Container';

function App() {
  return (
    <>
      <Navbar bg="dark" expand="lg" variant="dark">
        <Container>
          <Navbar.Brand href="/">My Blog</Navbar.Brand>
          <Navbar.Toggle aria-controls="basic-navbar-nav" />
          <Nav className="me-auto">
            <Nav.Link href="/posts/new">New</Nav.Link>
          </Nav>
        </Container>
      </Navbar>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/posts/:id" element={<Post />} />
        <Route path="/posts/new" element={<Create />} />
        <Route path="/posts/:id/edit" element={<Edit />} />
      </Routes>
    </>
  );
}

export default App;
```

### Axios client

Our client will have to make API calls to the server to perform operations on the database, this is why we installed `axios` at the beginning of this tutorial. It is now time to use it, however we can wrap inside a `http` library file and export it as a module:

1. We need to take into account that making API calls in local is like calling a different server as client and servers run on different ports, a complete different configuration compared to the deployment on Koyeb later on.
2. The HTTP object is exported along with the basic methods to call GET, POST, PUT and DELETE endpoints.

Create a new folder `/lib` and inside add `http.js`

```bash
  mkdir lib
  touch /lib/http.js
```

Add the following code:

```javascript
import axios from 'axios';
// When building the client into a static file, we do not need to include the server path as it is returned by it
const domain = process.env.NODE_ENV === 'production' ? '' : 'http://localhost:3001';

const http = (
  url,
  {
    method = 'GET',
    data = undefined,
  },
) => {
  return axios({
    url: `${domain}${url}`,
    method,
    data,
  });
};

// Main functions to handle different types of endpoints
const get = (url, opts = {}) => http(url, { ...opts });
const post = (url, opts = {}) => http(url, { method: 'POST', ...opts });
const put = (url, opts = {}) => http(url, { method: 'PUT', ...opts });
const deleteData = (url, opts = {}) => http(url, { method: 'DELETE', ...opts });

const methods = {
  get,
  post,
  put,
  delete: deleteData,
};

export default methods;
```

We will see how to use the `http` object in the next section.

### Create containers and reusable components

The very first component we are going to build is `Home`, in charge of rendering the list of posts as well as the header of the homepage.
To render the list of posts Home has to 
1. Call the server GET `/api/posts` endpoint after the first rendering
2. Store the array posts in the state
3. Render the posts to the user and link them to `/posts/:post_id` to read the content

In the terminal create a folder `/pages` and a file `home.js` in it:

```bash
mkdir pages
touch pages/home.js
```
Add the following code in it:

```javascript
import { useEffect, useState } from 'react';
// Link component allow users to navigate to the blog post component page
import { Link } from 'react-router-dom';
import Container from 'react-bootstrap/Container';
import ListGroup from 'react-bootstrap/ListGroup';
import Image from 'react-bootstrap/Image';
import http from '../lib/http';
// utility function to format the creation date
import formatDate from '../lib/formatDate';

const Home = () => {
  // useState allows us to make use of the component state to store the posts
  const [posts, setPosts] = useState([]); 
  useEffect(() => {
    // Call the server to fetch the posts and store them into the state
    async function fetchData() {
      const { data } = await http.get('/api/posts');
      setPosts(data.data.posts);
    }
    fetchData();
  }, []);
  
  return (
    <>
      <Container className="my-5" style={{ maxWidth: '800px' }}>
        <Image
          src="avatar.jpeg"
          width="150"
          style={{ borderRadius: '50%' }}
          className="d-block mx-auto img-fluid"
        />
        <h2 className="text-center">Welcome to the Digital Marketing blog</h2>
      </Container>
      <Container style={{ maxWidth: '800px' }}>
        <ListGroup variant="flush" as="ol">
          {
            posts.map((post) => {
              // Map the posts to JSX
              return (
                <ListGroup.Item key={post._id}> 
                  <div className="fw-bold h3">
                    <Link to={`/posts/${post._id}`} style={{ textDecoration: 'none' }}>{post.title}</Link>
                  </div>
                  <div>{post.author} - <span className="text-secondary">{formatDate(post.createdAt)}</span></div>
                </ListGroup.Item>
              );
            })
          }
        </ListGroup>
      </Container>
    </>
  );
};

export default Home;
```

One special mention goes to `formatDate`, a utility function that formats the post creation date to "MMMM DD, YYYY". We are expecing to call it in other components as well, this is why it is decoupled from `Home` into its own file.
In the terminal create the file `formatDate.js` under `/lib`:

```bash
touch lib/formatDate.js
```

```javascript
const formatDate = (date, locale = 'en-US') => {
  if (!date) return null;

  const options = { year: 'numeric', month: 'long', day: 'numeric' };
  const formattedDate = new Date(date);
  return formattedDate.toLocaleDateString(locale, options);
};

export default formatDate;
```

It is a very straightforward function that takes the date from the DB, create a `Date` object and format it by setting locale and options.

The resulting UI will look like this:

[react-homepage]

The logic behind showing the blog post content is not too different than the one we saw for `Home`:

1. When hitting `/posts/post_id` the client calls the server API to fetch the specific blog post.
2. The post is stored in the component state.
3. Using react-boostrap, we create a simple-but-effective UI for the users to read the post.
4. On top of this, we add 2 buttons to either "edit" or "delete" the posts. Specifically, "edit" is nothing more than a link to `/posts/post_id/edit` and delete calls DELETE `/api/posts/:post_id` and then redirect the user to the homepage.

Open the terminal and create `post.js` under `/pages`:

```bash
touch post.js
```

```javascript
import { useEffect, useState } from 'react';
import { useParams, useNavigate, Link } from 'react-router-dom';
import Container from 'react-bootstrap/Container';
import Button from 'react-bootstrap/Button';
import http from '../lib/http';
import formatDate from '../lib/formatDate';

const Post = () => {
  const { id: postId } = useParams();
  const [post, setPost] = useState({});
  const navigate = useNavigate();
  // Fetch the single blog post
  useEffect(() => {
    async function fetchData() {
      const { data } = await http.get(`/api/posts/${postId}`);
      setPost(data.data.post);
    }
    fetchData();
  }, [postId]);
  // Delete the post and redirect the user to the homepage
  const deletePost = async () => {
    await http.delete(`/api/posts/${postId}`);
    navigate('/');
  }
  
  
  return (
    <>
      <Container className="my-5 text-justified" style={{ maxWidth: '800px' }}>
        <h1>{post.title}</h1>
        <div className="text-secondary mb-4">{formatDate(post.createdAt)}</div>
        {post.tags?.map((tag) => <span>{tag} </span>)}
        <div className="h4 mt-5">{post.content}</div>
        <div className="text-secondary mb-5">- {post.author}</div>
        <div className="mb-5">
          <Link
            variant="primary"
            className=" btn btn-primary m-2"
            to={`/posts/${postId}/edit`}
          >
            Edit
          </Link>
          <Button variant="danger" onClick={deletePost}>Delete</Button>
        </div>
        <Link to="/" style={{ textDecoration: 'none' }}>&#8592; Back to Home</Link>
      </Container>
    </>
  );
};

export default Post;
```

The UI will look like this:

[react-post-content]

As we redirect the user to another page when editing the blog post, create the file `edit.js` inside `/pages`:

```bash
touch edit.js
```

The UI will show a form filled with the blog post data for title, author, content and tags. Users can
1. Edit each one of the fields
2. submit the data to the server by calling PUT `/api/posts/:post_id`

Note that we are using `react-hook-form` to register fields, collect the data and submit to the server. For the purpose of the tutorial we are not performing any validation on the data but it is fairly simple to be added thanks to react-hook-form simple API.

```javascript
import { useEffect } from 'react';
import { useParams, useNavigate, Link } from 'react-router-dom';
import { useForm } from 'react-hook-form';
import Container from 'react-bootstrap/Container';
import Button from 'react-bootstrap/Button';
import Form from 'react-bootstrap/Form';
import http from '../lib/http';

const Edit = () => {
  const { id: postId } = useParams();
  const navigate = useNavigate();
  const { register, handleSubmit, reset } = useForm();
  // we call the API to fetch the blog post current data
  useEffect(() => {
    async function fetchData() {
      const { data } = await http.get(`/api/posts/${postId}`);
      // by calling "reset", we fill the form fields with the data from the database
      reset(data.data.post);
    }
    fetchData();
  }, [postId, reset]);
  
  const onSubmit = async ({ title, author, tags, content }) => {
    const payload = {
      title,
      author,
      tags: tags.split(',').map((tag) => tag.trim()),
      content,
    };
    await http.put(`/api/posts/${postId}`, { data: payload });
    navigate(`/posts/${postId}`);
  };
  
  return (
    <Container className="my-5" style={{ maxWidth: '800px' }}>
      <h1>Edit your Post</h1>
      <Form onSubmit={handleSubmit(onSubmit)} className="my-5">
        <Form.Group className="mb-3">
          <Form.Label>Title</Form.Label>
          <Form.Control type="text" placeholder="Enter title" {...register('title')} />
        </Form.Group>
        <Form.Group className="mb-3">
          <Form.Label>Author</Form.Label>
          <Form.Control type="text" placeholder="Enter author" {...register('author')} />
        </Form.Group>
        <Form.Group className="mb-3">
          <Form.Label>Tags</Form.Label>
          <Form.Control type="text" placeholder="Enter tags" {...register('tags')} />
          <Form.Text className="text-muted">
            Enter them separately them with ","
          </Form.Text>
        </Form.Group>
        <Form.Group className="mb-3">
          <Form.Label>Content</Form.Label>
          <Form.Control as="textarea" rows={3} placeholder="Your content..." {...register('content')} />
        </Form.Group>
        <Button variant="primary" type="submit">Save</Button>
      </Form>
      <Link to="/" style={{ textDecoration: 'none' }}>&#8592; Back to Home</Link>
    </Container>
  );
};

export default Edit;
```

NB: With a centralized app state we would not need to call the API once again as we would have the post data already available in the client. However, in order not to add extra business logic to pass data on different views or handle refreshing the page, we simply call `/api/posts/post_id` once again.

Here is the page UI:

[react-edit-post]

The only missing action is giving users the ability of creating their own posts. We already created the button "New" in the navbar that redirects to `/posts/new`. 
Similarly to the previous page `edit.js`, we are prompting a form for the user to fill. Fields are initially empty as we are expecting to store a brand new blog post in the database.

Add a new file `create.js` in `/pages` and copy the following code:

```javascript
import { useNavigate, Link } from 'react-router-dom';
import { useForm } from 'react-hook-form';
import Container from 'react-bootstrap/Container';
import Button from 'react-bootstrap/Button';
import Form from 'react-bootstrap/Form';
import http from '../lib/http';

const Post = () => {
  const navigate = useNavigate();
  const { register, handleSubmit } = useForm();

  const onSubmit = async ({ title, author, tags, content }) => {
    const payload = {
      title,
      author,
      tags: tags.split(',').map((tag) => tag.trim()),
      content,
    };
    await http.post('/api/posts', { data: payload });
    navigate('/');
  };
  
  return (
    <Container className="my-5" style={{ maxWidth: '800px' }}>
      <h1>Create new Post</h1>
      <Form onSubmit={handleSubmit(onSubmit)} className="my-5">
        <Form.Group className="mb-3">
          <Form.Label>Title</Form.Label>
          <Form.Control type="text" placeholder="Enter title" {...register('title')} />
        </Form.Group>
        <Form.Group className="mb-3">
          <Form.Label>Author</Form.Label>
          <Form.Control type="text" placeholder="Enter author" {...register('author')} />
        </Form.Group>
        <Form.Group className="mb-3">
          <Form.Label>Tags</Form.Label>
          <Form.Control type="text" placeholder="Enter tags" {...register('tags')} />
          <Form.Text className="text-muted">
            Enter them separately them with ","
          </Form.Text>
        </Form.Group>
        <Form.Group className="mb-3">
          <Form.Label>Content</Form.Label>
          <Form.Control as="textarea" rows={3} placeholder="Your content..." {...register('content')} />
        </Form.Group>
        <Button variant="primary" type="submit">Publish</Button>
      </Form>
      <Link to="/" style={{ textDecoration: 'none' }}>&#8592; Back to Home</Link>
    </Container>
  );
};

export default Post;
```

The code is very similar to `edit.js`. Here is the UI:

[react-create-post]

Congratulations, we finished the UI as well so we are now ready to deploy our blog app on the internet!


## Deploy the app on Koyeb

Let's login on Koyeb and in the Control Panel click on the button "Create App".

We have a few fields to fill in:

1. In "Deployment method", choose Github.
2. Select the git repository and select the branch where you pushed the code to. In my case, `master`.
3. Enter the port 3001 we exposed from the server.
5. Add the build command `yarn build` as we need to build the client
6. Add MONGO_PWD as an environment variable with the password of the user you created on Mongo Atlas.
7. Name your application, `mern-blog`.

Once you clicked on "Create App", Koyeb will take care of deploying your app in just a few seconds and return a public URL to try the app.

Good job, we now have a usable blog app!

## Conclusions

In this first part of the series of the MERN web apps series, we built the basic blocks of a online Blog application. We initially setup a MOngoDB database hosted through Mongo Atlas, created an Express API server to fetch the data and a React client to show the data to the users. 
There are several enhancements we could on the client-side such as form validation, code refactoring etc. but the guide already covers the basic to complete every step of creating and deploy a MERN app.
We will see you soon on the second part where you are going to explore the search abilities of Mongo Atlas.

