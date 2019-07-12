# Next.js

This tutorial is based on Travery Media's Youtube video -> <a href="https://www.youtube.com/watch?v=IkOVe40Sy0U" target="blank">Click me</a>

Next is used to render server side React applications. Traditionally React is run on the browser, but Next lets us run React applications on the server.
It also gives you a lot of things straight out of the box, like routing!

## Core Features

- Server-rendered by default
- Automatic code splitting for faster page loads (if you have a module used in less than half of your pages, it won't go into the main javascript bundle)
- Simple client-side routing (page/file based, can access with a URL)
- Built in CSS Support
- Webpack-based dev environment = Hot Reloading
- Easy deployment using Zeit's Now hosting or use the static build files and host as you would normally

## Installation & Setup

`mkdir hello-next`
`cd hello-next`
`npm init -y`
`npm install --save react react-dom next`
`mkdir pages`

Add this to your scripts

```
"scripts": {
  "dev": "next",
  "build": "next build",
  "start": "next start"
}
```

## Let's get started!

We're going to build a tiny application that fetches Bitcoin data.

Go ahead and create an index.js file inside the pages directory. Think of this as our root component.

Add the following code to your index.js file.

```
export default () => (<div>Hello Next.js</div>)
```

`npm run dev`

Easy as that! Now let's create an about page. Simply create a new file in the pages folder called about.js.

Add the following code to your about.js file and voila, you can visit that page too!

```
export default () => (<div>About Next.js</div>)
```

### Routing

Routing is as simple as using a Link tag. Don't forget to import it first. Let's set up routing for our index page.

```
import Link from 'next/link'

const Index = () => (
  <div>
      <ul>
          <li><Link href="/"><a>Home</a></Link></li>
      </ul>
      <ul>
          <li><Link href="/about"><a>About</a></Link></li>
      </ul>
    <h1>Welcome to BitzPrice</h1>
  </div>
)

export default Index
```

Nice. You'll probably want to move that navbar into it's own component. Go ahead and create a components folder as you would with any React application and create a Navbar.js file. Your index.js should now look like this.

```
import Navbar from '../components/Navbar'

const Index = () => (
  <div>
      <Navbar />
    <h1>Welcome to BitzPrice</h1>
  </div>
)

export default Index
```

Great - much neater!

### Introducing style jsx

We'll be removing this later, but just know that we can style right within the component! This styling is scoped and will only pertain to elements within this component! CSS is component based in Next.

```
import Link from "next/link"

const Navbar = () => (
  <div>
    <ul>
      <li>
        <Link href="/">
          <a>Home</a>
        </Link>
      </li>
      <li>
        <Link href="/about">
          <a>About</a>
        </Link>
      </li>
    </ul>

    <style jsx>
      {`
        ul {
          background: #333;
          color: #fff;
          list-style: none;
          display: flex;
        }

        ul li {
          font-size: 18px;
          margin-right: 20px;
        }

        ul li a {
          color: #fff;
          text-decoration: none;
        }
      `}
    </style>
  </div>
)

export default Navbar
```

### Introducing Head

Head is something that Next provides us with. Let's change the page title using Head. Notice that the Navbar has moved to it's own Layout.js component.

```
import Head from "next/head"
import Navbar from "./Navbar"

const Layout = props => (
  <div>
      <Head>
          <title>BitzPrice</title>
      </Head>
    <Navbar />
    {props.children}
  </div>
)

export default Layout
```

Let's bring in Bootstrap to our application (a customized version from Bootswatch). Simply add a link tag inside the Head.

````
    <Head>
          <title>BitzPrice</title>
          <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bootswatch/4.3.1/cerulean/bootstrap.min.css" />
      </Head>
      ```
````

## Working with APIs

Now that we've got a good understanding of how Next works. Let's finish building out the application using fetch. Note that we'll be using the package 'isomorphic-unfetch' which let's us use fetch on both the client and server side.

`npm i isomorphic-unfetch`

Now let's use a lifecycle method that Next provides us, getInitialProps, to fetch some data from the <a href="https://www.coindesk.com/api" target="blank">Coindesk API.</a>

```
import fetch from "isomorphic-unfetch"
import Layout from "../components/Layout"

const Index = props => (
  <Layout>
    <div>
      <h1>Welcome to BitzPrice</h1>
      {props.bpi.time.updated}
    </div>
  </Layout>
)

Index.getInitialProps = async () => {
  const res = await fetch("https://api.coindesk.com/v1/bpi/currentprice.json")
  const data = await res.json()

  return {
    bpi: data
  }
}

export default Index
```

You now have access to the Bitcoin price data returned from Coindesk! Let's use that in our index.js. We'll also be adding a selector to toggle between the 3 different currencies the API sends back. This is mainly React stuff, so feel free to skip to the next section where we'll be deploying our Next application using Zeit's Now.

First let's create a Prices component so we don't make our index.js messy. Then simply pass in the data as props.

```
const Index = props => (
  <Layout>
    <div>
      <h1>Welcome to BitzPrice</h1>
      <p>Check current Bitcoin rate</p>
      <Prices bpi={props.bpi} />
    </div>
  </Layout>
)
```

Now let's code our Prices component.

```
class Prices extends React.Component {
  state = {
    currency: "USD"
  }

  render() {
    let list = ""

    if (this.state.currency === "USD") {
      list = (
        <li className="list-group-item">
          Bitcoin rate for {this.props.bpi.USD.description}:{" "}
          <span className="badge badge-primary">{this.props.bpi.USD.code}</span>{" "}
          <strong>{this.props.bpi.USD.rate}</strong>
        </li>
      )
    } else if (this.state.currency === "GBP") {
      list = (
        <li className="list-group-item">
          Bitcoin rate for {this.props.bpi.GBP.description}:{" "}
          <span className="badge badge-primary">{this.props.bpi.GBP.code}</span>{" "}
          <strong>{this.props.bpi.GBP.rate}</strong>
        </li>
      )
    } else if (this.state.currency === "EUR") {
      list = (
        <li className="list-group-item">
          Bitcoin rate for {this.props.bpi.EUR.description}:{" "}
          <span className="badge badge-primary">{this.props.bpi.EUR.code}</span>{" "}
          <strong>{this.props.bpi.EUR.rate}</strong>
        </li>
      )
    }

    return (
      <div>
        <ul className="list-group">{list}</ul>
        <br />
        <select
          onChange={e => this.setState({ currency: e.target.value })}
          className="form-control"
        >
          <option value="USD">USD</option>
          <option value="GBP">GBP</option>
          <option value="EUR">EUR</option>
        </select>
      </div>
    )
  }
}

export default Prices
```

## Deployment
We've built our application, now it's time to deploy and share it with the rest of the world!

-- Coming soon.

