# CS52 Workshops: Alternative Server-Side Generators

## Overview
Gatsby.js is a server-side static side generator built on top of React. This framework allows you to define individual site pages based on JS files and parses those into HTML at build-time. This solution is ideal for static content or pages that don't require extensive interaction with the user or external data sources, because it minimizes the amount of HTML that the client has to process. 

In this tutorial, we'll build a simple landing page using Gatsby.js and demonstrate how Gatsby makes building static sites in React far easier. 

## Setup
### Gatsby Installation
Begin by installing the Gatsby CLI. 

```npm install -g gatsby-cli```

Note: if you get a command not found error when running gatsby, try using the following command to install the CLI: ```sudo yarn global add gatsby-cli```


This will install Gatsby on your machine globally.

### Create a Site
1. Create a new site from the Gatsby starter pack: 

```gatsby new hello-world https://github.com/gatsbyjs/gatsby-starter-hello-world```

This command will create the backbone for your server, but don't worry – we'll add more to it.

Select yarn as your package manager if prompted. This process may take a while the first time. 

2. Navigate to your new directory. 

```cd hello-world```

3. Start your development server. 

```gatsby develop```

 This is the equivalent of ```yarn start``` and will serve your site on http://localhost:8000/ and enable hot reloading, like a normal React site. 

4. Open your site in your web browser. You should see a blank page with "Hello world!" on the screen. 

    ![](https://i.imgur.com/4zKxVIH.png)
    
5. Examine the files in your repository. Feel free to play around with the text in your index.js file.

    **Note:** Gatsby uses hot reloading, so updates to your index.js file will appear in real time. 


## Step by Step
### Building your Homepage
Now that you have a home page set up, let's add content and some styling.

1. Let's begin by changing the text in the page to Home Page.

```
import React from "react"

export default () => <div className="header">Home Page</div>
```

2. From here, it's helpful to use a global CSS stylesheet. Do this by adding an `src/style/global.css` file to your project.

```
cd src
mkdir styles
cd styles
touch global.css
```
3. Let's add some styling!
```
div {
    font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
}

html {
    background-color: rgb(250, 252, 252);
}

.header {
    font-weight: 500;
    font-size: 24px;
}

.content {
    color: rgba(0,0,0,0.7);
    line-height: 1.2;
}

img {
    width: 100%;
    height: 100%;
}
```

4. In your **main directory** (outside of the src folder), add a file named `gatsby-browser.js` and add an import statement to the top: `import "./src/styles/global.css"`. This line will tell Gatsby to use the stylesheet you just created when building your website. 

5. Restart your local development server to commit the changes. 


### Adding Another Page
Now, let's create a page that programatically expresses data. One advantage of Gatsby is that you can easily separate *structure* and *style* from *content*. You can define the layout of each of your pages and import the content from external data sources, like Markdown files or an API.

1. Create a new file in `src/pages` called `about.js`.
2. Add some boilerplate to `about.js`. Don't worry, we'll fill this in later!

```
import React from "react"; 

const About = () => {
    return (
        <div>
            <div className="header"></div>
            <div className="content"></div>
        </div>
    ); 
}

export default About; 
```

3. Run the following in your command line: `yarn add gatsby-source-filesystem gatsby-transformer-remark`

    This command adds the plugins to read files from the filesystem and transform Markdown files into HTML.
4. Replace the contents of module.exports in `gatsby-config.js` with the following:

```
  plugins: [
    {
      resolve: `gatsby-source-filesystem`,
      options: {
        path: `${__dirname}/src/markdown-pages`,
        name: `markdown-pages`,
      },
    },
    `gatsby-transformer-remark`,
  ],
```
This configures the plugins you just added with Gatsby. 

5. Create a directory at `src/markdown-pages`, and add `about.md` to this folder.
6. Let's fill in `about.md` now.
```
---
path: "/about"
title: "About"
---

Here's some content that tells you a little about this page. Gatsby will automatically convert *everything* in here to HTML!

![](http://cs52.me/assets/imgs/cs52logo.png)
```
Everything demarcated within the triple dashes is called *frontmatter*. Frontmatter is a set of key-value pairs that can be passed around along with the content. The value assigned to `path` will be used to link your JS page with this Markdown content.

#### GraphQL
At this point, it's worth unpacking GraphQL a little. GraphQL is a query language – it's used to retrieve data in a particular JSON format. You can query a data source using GraphQL and receive data in the exact shape you asked for it – no more navigating rows or columns in a SQL database!

This allows us to build sites from data sources such as Markdown, WordPress, etc. The data layer is powered by GraphQL. GraphQL pulls data at *build-time* – unlike a normal React app that pulls data on the client's side. 

You can go to `http://localhost:8000/___graphql` to explore your site's existing data and schema. As you can see, even right out of the box, you have access to a lot of information from a GraphQL query. 

Let's integrate GraphQL into our About page.

7. Replace `about.js` with the following:

```
import React from "react"; 
import { graphql } from "gatsby"; 

export default function About({ data }) {
    const { markdownRemark } = data
    const { frontmatter, html } = markdownRemark

    return (
        <div>
            <div className="header">{frontmatter.title}</div>
            <div className="content" dangerouslySetInnerHTML={{ __html: html }}></div>
        </div>
    ); 
}

export const pageQuery = graphql`
    query AboutQuery {
        markdownRemark(fileAbsolutePath: { regex: "/markdown-pages/about.md/" }) {
            html
            frontmatter {
                title
            }
        }
    }
`
```
At the bottom, we've added a GraphQL query. This query will retrieve the contents of about.md at build-time and fill in our `data` prop and HTML elements accordingly. 

Note that while this example was rather simple, GraphQL queries can be expanded to encompass anything. For instance, imagine constructing a single blog post template and then automatically generating new post pages with each Markdown file you add. 

Here's what it should look like: 
![](https://i.imgur.com/Vy8fgdT.png)


### Navigation Bar
Let’s say the homepage and the about page both get quite large and you have to rewrite a lot of things. You can use sub-components to break the UI into reusable pieces. Both of your pages have a navigation bar — let's create a component that will describe this.

To do this, we'll build a Gatsby *layout component.* Layout components are for sections of your site that you want to share across multiple pages.

1. Create a new directory at ```src/components```. 
2. Make a ```navbar.js``` file in that directory, and add the following code:
```
import React from "react"
import { Link } from "gatsby"

export default function Navbar({ children }) {
    return (
        <div style={{ maxWidth: 650, margin: '0 auto' }}>
            <div>
                <Link to="/">Home</Link>
                <Link to="/about/">About</Link>
            </div>
            {children}
        </div>
      )
}
```

Here, we're creating a layout component. Any page that incorporates this component will now receive the *global layout styling* defined in the parent div and will have the nav bar automatically added as well. 

`Link` is a special Gatsby class used for both internal and programmatic navigation. Link allows you to deploy *preloading*, which is used to pre-fetch resources so they're loaded before the user navigates to the page. That makes your websites super fast!

3. Modify the about and index page created in the previous section(s) to include the navigation: ```import Navbar from "../components/navbar"```. To incorporate a layout component, wrap what you're returning in the layout component, like so (`about.js`):

```
return (
        <Navbar>
            <div>
                <div className="header">{frontmatter.title}</div>
                <div className="content" dangerouslySetInnerHTML={{ __html: html }}></div>
            </div>
        </Navbar>
    ); 
```

<details>
<summary>And, index.js.</summary>

```
import React from "react"
import Navbar from "../components/navbar";

export default () => {
    return (
        <Navbar>
            <div className="header">Home Page</div>
        </Navbar>
    );
}
```

</details>


4. Let's style our nav bar now. Add the classes 'navbar-container' and 'navbar-link'. 

<details>
<summary>Here's what it should look like.</summary>

```
import React from "react"
import { Link } from "gatsby"

export default function Navbar({ children }) {
    return (
        <div style={{ maxWidth: 650, margin: '0 auto' }}>
            <div className="navbar-container">
                <Link to="/" className="navbar-link">Home</Link>
                <Link to="/about/" className="navbar-link">About</Link>
            </div>
            {children}
        </div>
      )
}
```

</details>

5. Navigate to your global.scss file, and add the following:
```
.navbar-container {
    display: flex;
    flex-direction: row;
    justify-content: space-around;
    margin-top: 20px;
    padding-bottom: 20px;
    margin-bottom: 20px;
    border-bottom: 1px solid rgba(0,0,0,0.3);
}

.navbar-link {
    text-decoration: none;
    color: rgba(0,0,0,0.8);
    font-size: 20px;
}

a:hover {
    color: rgb(255, 187, 85); 
}
```
6. Congrats! Your navbar is now functional.

![](https://i.imgur.com/twXGyiT.png)

## Summary / What You Learned
* [ ] Built a static site in Gatsby
* [ ] Used GraphQL to query Markdown files for data
* [ ] Created a layout component for a navigation bar
* [ ] Used the Link component to connect pages with pre-loading

You now know how to use another static site generator! Why is this cool?

Gatsby can be used to build static sites that are Progressive Web Apps, follow the latest web standards, and are optimized to be highly performant. It makes use of the latest and popular technologies including ReactJS, Webpack, GraphQL, modern ES6+ JavaScript and CSS. In other words - everything that we have been using in this class.

This is important becuase there is no reason to use complicated frameworks such as create-react-app in order to build static websites, as developers who are not familiar with newer stacks, or people new to programming websites, will have a hard time following the project structure. However Gatsby circumvents these complicated structures, while making use of all the new cool technology we have been using in this class. 

## Reflection

1. Think of an example where a developer would rather build a static site rather than a dynamic site, and vice versa.
2. Take a look at this site that uses Gatsby - https://airbnb.design/cereal/
   Now open a famous dynamic website - https://www.amazon.com/

   Why is one opening new pages faster than the other? (Hint - one is running a lot more scripts.)


## Resources
1. Gatsby.js: https://www.gatsbyjs.org/
2. Gatsby tutorial: https://www.gatsbyjs.org/tutorial/
