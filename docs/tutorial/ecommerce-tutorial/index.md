---
title: "Gatsby E-Commerce Tutorial"
---

In this advanced tutorial, you’ll learn how to use Gatsby to build the UI for a basic e-commerce site that can accept payments, with [Stripe](https://stripe.com) as the backend for processing payments.

## Why use Gatsby for an e-commerce site?

Benefits of using Gatsby for e-commerce sites include the following:

- Security inherent in static sites
- Blazing fast performance when your pages are converted from React into static files
- Easy to host

You can see the working demo hosted here: https://gatsby-ecommerce.netlify.com/

## Prerequisites

- Since this is a more advanced tutorial, building a site with Gatsby before will likely make this tutorial less time-consuming ([see the main tutorial here](/tutorial/))
- Stripe account: [register for an account here](https://dashboard.stripe.com/register)

### How does Gatsby work with Stripe?

Stripe is a payment processing service that allows you to securely collect and process payment information from your customers. To try out Stripe for yourself, go to [Stripe’s Quick Start Guide](https://stripe.com/docs/payments/checkout#tryout).

There are alternatives to Stripe, like Square and Braintree, and their setup is very similar to Stripe.

Stripe offers a [hosted checkout](https://stripe.com/docs/payments/checkout) that doesn't require any backend component. You can configure products, SKUs, and subscription plans in the [Stripe Dashboard](https://stripe.com/docs/payments/checkout#configure). If you're selling a single product or subscription (like an eBook) you can hardcode the product's SKU ID in your Gatsby side. If you're selling multiple products, you can use the [Stripe source plugin](https://www.gatsbyjs.org/packages/gatsby-source-stripe/) to retrieve all SKUs at build time. If you want your Gatsby site to automatically update, you can use the Stripe webhook event to [trigger a redeploy](https://www.netlify.com/docs/webhooks/) when a new product or SKU is added.

> **NOTE**: Stripe Checkout is currently in beta. You can sign up to receive updates on the [Stripe website](https://stripe.com/docs/payments/checkout). In the meantime, if you're looking to build more custom checkout flows, you might need to set up a simple function that your Gatsby project can POST to in order to handle the payment. See the ["custom example"](#custom-fully-custom-checkout-flow-requires-backend-component) section below for more details.

## Setting up a Gatsby site

Create a new Gatsby project by running the `gatsby new` command in the terminal and change directories into the new project you just started:

```shell
gatsby new ecommerce-gatsby-tutorial
cd ecommerce-gatsby-tutorial
```

## Installing the StripeJS plugin

You can extend the functionality of this default starter with plugins. One such plugin is `gatsby-plugin-stripe`, which you’ll install in this project:

```shell
npm install gatsby-plugin-stripe
```

Open the root site directory in a text editor and navigate to `gatsby-config.js` and add the StripeJS plugin to `gatsby-config.js` in the plugins section. Your `gatsby-config.js` should look like the following code example:

```js:title=gatsby-config.js
module.exports = {
  siteMetadata: {
    title: "Gatsby E-Commerce Starter",
  },
  plugins: ["gatsby-plugin-react-helmet", "gatsby-plugin-stripe"],
}
```

### See your site hot reload in the browser!

Run `npm run develop` in the terminal, which starts a development server and reloads changes you make to your site so you can preview them in the browser. Open up your browser to [localhost:8000](http://localhost:8000/) and you should see a default homepage.

> **NOTE**: If you have already started your Gatsby development server using `npm run develop`, you will need to restart the server by pressing CTRL + C in the terminal where the command was run and running `npm run develop` again to see changes in your `gatsby-config.js` reflected on [localhost:8000](http://localhost:8000/)

### How does the StripeJS plugin work?

Stripe provides a JavaScript library the allows you to securely redirect your customer to the Stripe hosted checkout page. The Gatsby plugin, `gatsby-plugin-stripe`, will add this snippet:

```html
<script src="https://js.stripe.com/v3/"></script>
```

to the end of the `<body>` tag across all of your pages. This helps facilitate Stripe's [fraud detection](https://stripe.com/docs/stripe-js/reference#including-stripejs).

If you want to further customise the checkout process or pull Stripe data into your site, check out [Gatsby's plugin library for more Stripe plugins](https://www.gatsbyjs.org/plugins/?=stripe).

### Getting your Stripe test keys

View your API credentials by logging into your Stripe account, and then going to Developers > API Keys.

![Stripe public test key location in Stripe account](stripe-public-test-key.png)

You have 2 keys in both test mode and production mode:

- a publishable key
- a secret key

While testing, you must use the key(s) that include _test_. For production code, you will need to use the live keys. As the names imply, your publishable key may be included in code that you share publicly (for example, on the frontend, and in GitHub), whereas your secret key should not be shared with anyone or committed to any public repo. It’s important to restrict access to this secret key because anyone who has it could potentially read or send requests from your Stripe account and see information about charges or purchases or even refund customers.

## Examples

You can find an implementation of these examples [on GitHub](https://github.com/thorsten-stripe/ecommerce-gatsby-tutorial).

### Easy: One Button

If you're selling a simple product, like an eBook for example, you can create a single button that will perform a redirect to the Stripe Checkout page:

#### Create a product and SKU

For Stripe Checkout to work without any backend component, you need to create a product listing in the Stripe Dashboard. This is required for Stripe to validate that the request coming from the frontend is legitimate and to charge the right amount for the selected product/SKU. To set this up, simply follow the steps in the [Stripe docs](https://stripe.com/docs/payments/checkout#configure).

#### Create a checkout component that loads StripeJS and redirects to the checkout

Create a new file at `src/components/checkout.js`. Your `checkout.js` file should look like this:

```jsx:title=src/components/checkout.js
import React from "react"

const buttonStyles = {
  fontSize: "13px",
  textAlign: "center",
  color: "#fff",
  outline: "none",
  padding: "12px 60px",
  boxShadow: "2px 5px 10px rgba(0,0,0,.1)",
  backgroundColor: "rgb(255, 178, 56)",
  borderRadius: "6px",
  letterSpacing: "1.5px",
}

const Checkout = class extends React.Component {
  // Initialise Stripe.js with your publishable key.
  // You can find your key in the Dashboard:
  // https://dashboard.stripe.com/account/apikeys
  componentDidMount() {
    this.stripe = window.Stripe("pk_test_jG9s3XMdSjZF9Kdm5g59zlYd", {
      betas: ["checkout_beta_4"],
    })
  }

  async redirectToCheckout(event) {
    event.preventDefault()
    const { error } = await this.stripe.redirectToCheckout({
      items: [{ sku: "sku_DjQJN2HJ1kkvI3", quantity: 1 }],
      successUrl: `http://localhost:8000/page-2/`,
      cancelUrl: `http://localhost:8000/`,
    })

    if (error) {
      console.warn("Error:", error)
    }
  }

  render() {
    return (
      <button
        style={buttonStyles}
        onClick={event => this.redirectToCheckout(event)}
      >
        BUY MY BOOK
      </button>
    )
  }
}

export default Checkout
```

#### What did you just do?

You imported React, added a button with some styles, and introduced some React functions. The `componentDidMount()` and `redirectToCheckout()` functions are most important for the Stripe functionality. The `componentDidMount()` function is a React lifecycle method that launches when the component is first mounted to the DOM, making it a good place to initialise the Stripe.js client. It looks like this:

```js:title=src/components/checkout.js
  componentDidMount() {
    this.stripe = window.Stripe('pk_test_jG9s3XMdSjZF9Kdm5g59zlYd', {
      betas: ['checkout_beta_4'],
    })
  }
```

This identifies you with the Stripe platform, validates the checkout request against your products and security settings, and processes the payment on your Stripe account.

```js:title=src/components/checkout.js
  async redirectToCheckout(event) {
    event.preventDefault()
    const { error } = await this.stripe.redirectToCheckout({
      items: [{ sku: 'sku_DjQJN2HJ1kkvI3', quantity: 1 }],
      successUrl: `http://localhost:8000/page-2/`,
      cancelUrl: `http://localhost:8000/`,
    })

    if (error) {
      console.warn('Error:', error)
    }
  }
```

The `redirectToCheckout()` function validates your checkout request and either redirects to the Stripe hosted checkout page or resolves with an error object. Make sure to replace `successUrl` and `cancelUrl` with the appropriate URLs for your application.

```js:title=src/components/checkout.js
  render() {
    return (
      <button
        style={buttonStyles}
        onClick={event => this.redirectToCheckout(event)}
      >
        BUY MY BOOK
      </button>
    )
  }
```

The `render()` function applies our styles to the button and binds the `redirectToCheckout()` function to the button's onclick event.

#### Importing the checkout component into the homepage

Now go to your `src/pages/index.js` file. This is your homepage that shows at the root URL. Import your new checkout component in the file underneath the other imports and add your `<Checkout />` component within the `<Layout>` element. Your `index.js` file should now look like similar to this:

```js:title=src/pages/index.js
import React from "react"
import { Link } from "gatsby"

import Layout from "../components/layout"
import Image from "../components/image"
import SEO from "../components/seo"

import Checkout from "../components/checkout" // highlight-line

const IndexPage = () => (
  <Layout>
    <SEO title="Home" keywords={[`gatsby`, `application`, `react`]} />
    <h1>Hi people</h1>
    <p>Welcome to your new Gatsby site.</p>
    <p>Now go build something great.</p>
    <Checkout /> {/* highlight-line */}
    <div style={{ maxWidth: `300px`, marginBottom: `1.45rem` }}>
      <Image />
    </div>
    <Link to="/page-2/">Go to page 2</Link>
  </Layout>
)

export default IndexPage
```

If you go back to [localhost:8000](http://localhost:8000/) in your browser and you have `npm run develop` running, you should now see a big, enticing "BUY MY BOOK" button. C'mon and give it a click!

### Advanced: Import SKUs via source plugin

Instead of hardcoding the SKU IDs, you can use the [gatsby-source-stripe plugin](https://www.gatsbyjs.org/packages/gatsby-source-stripe/) to retrieve your SKUs at build time.

#### Add the Stripe source plugin

Add the [gatsby-source-stripe plugin](https://www.gatsbyjs.org/packages/gatsby-source-stripe/) which you can use to pull in the SKUs from your Stripe account.

```shell
npm install gatsby-source-stripe
```

Now you can add the plugin configuration in your `gatsby-config` file:

```js:title=gatsby-config.js
module.exports = {
  siteMetadata: {
    title: `Gatsby E-Commerce Starter`,
  },
  plugins: [
    `gatsby-plugin-react-helmet`,
    "gatsby-plugin-stripe",
    {
      resolve: `gatsby-source-stripe`,
      options: {
        objects: ["Sku"],
        secretKey: process.env.STRIPE_SECRET_KEY,
        downloadFiles: true,
      },
    },
  ],
}
```

To retrieve your SKUs from your Stripe account you will need to provide your secret API key. This key needs to kept secret and must never be shared on the frontend or on GitHub. Therefore we need to set an environment variable to store the secret key. You can read more about the usage of env variables in Gatsby [here](https://www.gatsbyjs.org/docs/environment-variables/).

In the root directory of your project add a `.env.development` file:

```text:title=.env.development
# Stripe secret API key
STRIPE_SECRET_KEY=sk_test_xxx
```

To use the defined env variable you need to require it in your `gatsby-config.js` or `gatsby-node.js` like this:

```js:title=gatsby-config.js
require("dotenv").config({
  path: `.env.${process.env.NODE_ENV}`,
})
```

Lastly, make sure that your `.gitignore` file excludes all of your `.env.*` files:

```text:title=.gitignore
# dotenv environment variables files
.env
.env.development
.env.production
```

#### Create a component that lists your SKUs

In your components folder add a new `Products` folder. This folder will include the components that interact with the Stripe SKUs. First, you need a component that queries and lists your SKUs:

```jsx:title=src/components/Products/Skus.js
import React from "react"
import { graphql, StaticQuery } from "gatsby"

export default props => (
  <StaticQuery
    query={graphql`
      query SkusForProduct {
        skus: allStripeSku {
          edges {
            node {
              id
              currency
              price
              attributes {
                name
              }
            }
          }
        }
      }
    `}
    render={({ skus }) => (
      <div>
        {skus.edges.map(({ node: sku }) => (
          <p key={sku.id}>{sku.attributes.name}</p>
        ))}
      </div>
    )}
  />
)
```

You can validate your query and see what data is being returned in GraphiQL, which is available at http://localhost:8000/___graphql when running `npm run develop`.

Once you're happy with your query, create a new page where you can import the newly created Sku component:

```jsx:title=src/pages/advanced.js
import React from "react"
import { Link } from "gatsby"

import Layout from "../components/layout"
import SEO from "../components/seo"

import Skus from "../components/Products/Skus" // highlight-line

const AdvancedExamplePage = () => (
  <Layout>
    <SEO title="Advanced Example" />
    <h1>This is the advanced example</h1>
    <Skus /> {/* highlight-line */}
  </Layout>
)

export default AdvancedExamplePage
```

When navigating to http://localhost:8000/advanced/ you should now see a list of paragraphs with your SKU names.

#### Create a component that presents a single SKU

To make your SKUs more visually appealing and interactive, create a new `SkuCard` component in your `Products` folder:

```jsx:title=src/components/Products/SkuCard.js
import React from "react"

const cardStyles = {
  display: "flex",
  flexDirection: "column",
  justifyContent: "space-around",
  alignItems: "flex-start",
  padding: "1rem",
  marginBottom: "1rem",
  boxShadow: "5px 5px 25px 0 rgba(46,61,73,.2)",
  backgroundColor: "#fff",
  borderRadius: "6px",
  maxWidth: "300px",
}
const buttonStyles = {
  fontSize: "13px",
  textAlign: "center",
  color: "#fff",
  outline: "none",
  padding: "12px",
  boxShadow: "2px 5px 10px rgba(0,0,0,.1)",
  backgroundColor: "rgb(255, 178, 56)",
  borderRadius: "6px",
  letterSpacing: "1.5px",
}

const formatPrice = (amount, currency) => {
  let price = (amount / 100).toFixed(2)
  let numberFormat = new Intl.NumberFormat(["en-US"], {
    style: "currency",
    currency: currency,
    currencyDisplay: "symbol",
  })
  return numberFormat.format(price)
}

const SkuCard = class extends React.Component {
  async redirectToCheckout(event, sku, quantity = 1) {
    event.preventDefault()
    const { error } = await this.props.stripe.redirectToCheckout({
      items: [{ sku, quantity }],
      successUrl: `http://localhost:8000/page-2/`,
      cancelUrl: `http://localhost:8000/advanced`,
    })

    if (error) {
      console.warn("Error:", error)
    }
  }

  render() {
    const sku = this.props.sku
    return (
      <div style={cardStyles}>
        <h4>{sku.attributes.name}</h4>
        <p>Price: {formatPrice(sku.price, sku.currency)}</p>
        <button
          style={buttonStyles}
          onClick={event => this.redirectToCheckout(event, sku.id)}
        >
          BUY ME
        </button>
      </div>
    )
  }
}

export default SkuCard
```

This component renders a neat card for each individual SKU, with the SKU name, nicely formatted pricing, and a "BUY ME" button. The button triggers the `redirectToCheckout()` function with the corresponding SKU ID.

Lastly, we need to refactor our `Skus` component to initialise the Stripe.js client, and render `SkuCards` while handing down the Stripe.js client in the `props`:

```jsx:title=src/components/Products/Skus.js
import React, { Component } from 'react'
import { graphql, StaticQuery } from 'gatsby'
import SkuCard from './SkuCard' // highlight-line

const conatinerStyles = {
  display: 'flex',
  flexDirection: 'row',
  flexWrap: 'wrap',
  justifyContent: 'space-between',
  padding: '1rem 0 1rem 0',
}

class Skus extends Component {
  // Initialise Stripe.js with your publishable key.
  // You can find your key in the Dashboard:
  // https://dashboard.stripe.com/account/apikeys
  // highlight-start
  state = {
    stripe: window.Stripe('pk_test_qOUT7QyfVVOJlWXtnbzzZ9Tn', {
      betas: ['checkout_beta_4'],
    }),
  }
  // highlight-end

  render() {
    return (
      <StaticQuery
        query={graphql`
          query SkusForProduct {
            skus: allStripeSku {
              edges {
                node {
                  id
                  currency
                  price
                  attributes {
                    name
                  }
                }
              }
            }
          }
        `}
        render={({ skus }) => (
          <div style={conatinerStyles}>
            {skus.edges.map(({ node: sku }) => (
              <SkuCard key={sku.id} sku={sku} stripe={this.state.stripe} /> {/* highlight-line */}
            ))}
          </div>
        )}
      />
    )
  }
}

export default Skus
```

#### Adding a cart component

You can call `redirectToCheckout()` providing an array of SKUs and their quantities to charge for multiple items at the same time. Instead of each "BUY ME" button redirecting to the checkout page, you can therefore provide a central "GO TO CHECKOUT" button that uses the state of a cart component. You can see the necessary changes for this example [on GitHub](https://github.com/thorsten-stripe/ecommerce-gatsby-tutorial/tree/cart-example).

### Custom: Fully custom checkout flow (requires backend component)

Stripe Checkout is currently in beta. You can sign up to receive updates on the [Stripe website](https://stripe.com/docs/payments/checkout). In the meantime, if you're looking to build more custom checkout flows, you can set up a simple function that your Gatsby project can POST to in order to handle the payment. See the previous version of [this tutorial](https://github.com/gatsbyjs/gatsby/blob/6b3c08782d0898719b61181638b6a0967da49dd6/docs/tutorial/ecommerce-tutorial/index.md) for detailed steps.

## Testing Payments

In test mode (when using the API key that includes _test_) Stripe provides [test cards](https://stripe.com/docs/testing#cards) for you to test different checkout scenarios.
