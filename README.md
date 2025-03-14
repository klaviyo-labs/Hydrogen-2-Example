## Active on Site tracking
With Hydrogen 2, you will want to import script with import {Script} from ‘@shopify/hydrogen’; in root.jsx and add our standard onsite snippet within a Script element.  Replace the placeholder value for the company_id in the example below with your Klaviyo Public ID found under https://www.klaviyo.com/settings/account/api-keys:
```
export default function App() {
  const nonce = useNonce();
  /** @type {LoaderReturnData} */
  const data = useLoaderData();

  return (
    <html lang="en">
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width,initial-scale=1" />
        <Meta />
        <Links />
      </head>
      <body>
        <Layout {...data}>
          <Outlet />
        </Layout>
        <Script async type="text/javascript"
  src="//static.klaviyo.com/onsite/js/klaviyo.js?company_id=YourPublicKey" />
        <ScrollRestoration nonce={nonce} />
        <Scripts nonce={nonce} />
      </body>
    </html>
  );
}
```

## Viewed Product and Added to Cart tracking

To implement viewed product and added to cart tracking, we recommend first creating a component for our onsite tracking.  In our example, we've created an Onsite component in app/components/Onsite.jsx and add our standard viewed product and atc tracking requests:

```
export function trackViewedProduct(product) {
    let klaviyo = window.klaviyo || [];
    let item = {
        Name: product.title,
        ProductID: product.id.substring(product.id.lastIndexOf('/') + 1),
        ImageURL: product.selectedVariant.image.url,
        Handle: product.handle,
        Brand: product.vendor,
        Price: product.selectedVariant.price.amount,
        Metadata: {
          Brand: product.vendor,
          Price: product.selectedVariant.unitPrice,
          CompareAtPrice: product.selectedVariant.compareAtPrice,
        }
};
klaviyo.push(['track', 'Hydrogen Viewed Product', item]);
klaviyo.push(['trackViewedItem', item]);


}

export function trackAddedToCart(product) {
let klaviyo = window.klaviyo || []
let item = {
        Name: product.title,
        ProductID: product.id.substring(product.id.lastIndexOf('/') + 1),
        ImageURL: product.selectedVariant.image.url,
        Handle: product.handle,
        Brand: product.vendor,
        Price: product.selectedVariant.price.amount
      }
      klaviyo.push(['track', 'Hydrogen Added To Cart', item])
}
```

##### Import Functions

You can now import those functions into your product file (example app/routes/product.handle.jsx).  Import the onsite functions and useEffect into your product file:
```
import { useEffect } from "react";
import {trackViewedProduct, trackAddedToCart} from '~/components/Onsite';
```

##### Add Viewed Product

Load trackViewedProduct on page load via useEffect within your Product() function. Here’s an example of how this would look within your products.$handle.jsx file:

```

export default function Product() {
  /** @type {LoaderReturnData} */
  const {product, variants} = useLoaderData();
  const {selectedVariant} = product;

  // Execute VP on page load ***

  useEffect(() => {
    trackViewedProduct(product);
  },[]);


  return ( ...


```

##### Invoke Added To Cart Function

For trackAddedToCart, you’ll want to invoke this within your added to cart click handler. Here’s an example snippet using the ProductForm() function:

```

function ProductForm({product, selectedVariant, variants}) {

  // define atc ***
  const handleAtc = function () {
    trackAddedToCart(product)
  }

  return (
    <div className="product-form">
      <VariantSelector
        handle={product.handle}
        options={product.options}
        variants={variants}
      >
        {({option}) => <ProductOptions key={option.name} option={option} />}
      </VariantSelector>
      <br />
      <AddToCartButton
        disabled={!selectedVariant || !selectedVariant.availableForSale}
        onClick={() => {
          // Add our newly created click handler ***
          handleAtc();
          window.location.href = window.location.href + '#cart-aside';
        }}
        lines={
          selectedVariant
            ? [
                {
                  merchandiseId: selectedVariant.id,
                  quantity: 1,
                },
              ]
            : []
        }
      >
        {selectedVariant?.availableForSale ? 'Add to cart' : 'Sold out'}
      </AddToCartButton>
    </div>
  );
}

```

##### Additional Notes

Forms are supported for Hydrogen and other single paged applications, however functionality is limited.  Device display, geolocation and most visitor behavior that can be evaluated on initial page load should work as expected.  Behaviors like show/don't show on specific URL's are not supported as we run these evaluations when the script is executed (on the first page load); toggling between pages/views will not be factored into that logic for SPA's.  

With our onsite, forms should appear on local dev environments without any issues however Content Security Policies can impact the functionality of our scripts both on local and prod environments.  We suggest
testing through dev/preview builds sent up to Shopify via oxygen or custom deployments.  You will need to configure your CSP to your preference to allow our JS to function.

Example code can be referenced in the repository and the dev build can be [viewed here.](https://hydrogen2example-3ad64e75e88dc3df299c.o2.myshopify.dev/)
