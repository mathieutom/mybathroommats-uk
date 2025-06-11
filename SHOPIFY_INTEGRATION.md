# Shopify Integration Guide 🛒

This guide explains how to connect your bathroom mats e-commerce application to a real Shopify store.

## 🚀 Quick Setup

1. **Set up Shopify Storefront API access**
2. **Replace mock data with real API calls**
3. **Add environment variables**
4. **Test the integration**

## 📋 Prerequisites

- Shopify store (any plan)
- Storefront API access
- Basic knowledge of GraphQL

## 🔑 Step 1: Enable Storefront API

### In your Shopify Admin:

1. Go to **Apps** → **Manage private apps**
2. Click **Create private app**
3. Fill in app details:
   - **App name**: "My Bathroom Mats Website"
   - **Emergency developer email**: your-email@domain.com

### Set Permissions:

- **Storefront API**: ✅ Allow this app to access your storefront data
- **Read products**: ✅ Enabled
- **Read product listings**: ✅ Enabled
- **Read inventory**: ✅ Enabled
- **Read orders**: ✅ Enabled (if needed)

### Save and Copy:

- **Storefront access token**: Copy this token
- **Store URL**: your-store.myshopify.com

## 🔧 Step 2: Install Shopify SDK

```bash
npm install @shopify/storefront-api-client
```

## 📝 Step 3: Update Environment Variables

Create `.env` file in your project root:

```env
# Shopify Configuration
VITE_SHOPIFY_STORE_URL=your-store.myshopify.com
VITE_SHOPIFY_STOREFRONT_TOKEN=your_storefront_access_token_here
VITE_SHOPIFY_API_VERSION=2024-01

# Optional: For development
VITE_DEV_MODE=true
```

## 🔄 Step 4: Replace Mock Data

Update `src/lib/shopify.ts`:

<function_calls>
<invoke name="write_file">
<parameter name="title">Create real Shopify integration filee height
}
}
}
variants(first: 100) {
edges {
node {
id
title
price {
amount
currencyCode
}
compareAtPrice {
amount
currencyCode
}
availableForSale
quantityAvailable
selectedOptions {
name
value
}
image {
id
url
altText
width
height
}
}
}
}
options {
id
name
values
}
}
}
}
}
`;

const PRODUCT_BY_HANDLE_QUERY = `  query getProductByHandle($handle: String!) {
    productByHandle(handle: $handle) {
      id
      title
      description
      handle
      vendor
      productType
      tags
      createdAt
      updatedAt
      availableForSale
      priceRange {
        minVariantPrice {
          amount
          currencyCode
        }
        maxVariantPrice {
          amount
          currencyCode
        }
      }
      compareAtPriceRange {
        minVariantPrice {
          amount
          currencyCode
        }
        maxVariantPrice {
          amount
          currencyCode
        }
      }
      featuredImage {
        id
        url
        altText
        width
        height
      }
      images(first: 10) {
        edges {
          node {
            id
            url
            altText
            width
            height
          }
        }
      }
      variants(first: 100) {
        edges {
          node {
            id
            title
            price {
              amount
              currencyCode
            }
            compareAtPrice {
              amount
              currencyCode
            }
            availableForSale
            quantityAvailable
            selectedOptions {
              name
              value
            }
            image {
              id
              url
              altText
              width
              height
            }
          }
        }
      }
      options {
        id
        name
        values
      }
      metafields(identifiers: [{namespace: "custom", key: "features"}, {namespace: "custom", key: "care_instructions"}]) {
        key
        value
        type
        namespace
      }
    }
  }`;

const COLLECTIONS_QUERY = `  query getCollections($first: Int!) {
    collections(first: $first) {
      edges {
        node {
          id
          title
          description
          handle
          image {
            id
            url
            altText
            width
            height
          }
          products(first: 250) {
            edges {
              node {
                id
                handle
              }
            }
          }
        }
      }
    }
  }`;

// Transform Shopify data to our format
function transformShopifyProduct(shopifyProduct: any): Product {
return {
id: shopifyProduct.id,
title: shopifyProduct.title,
description: shopifyProduct.description || '',
price: parseFloat(shopifyProduct.priceRange.minVariantPrice.amount),
compareAtPrice: shopifyProduct.compareAtPriceRange.minVariantPrice?.amount
? parseFloat(shopifyProduct.compareAtPriceRange.minVariantPrice.amount)
: undefined,
handle: shopifyProduct.handle,
vendor: shopifyProduct.vendor,
productType: shopifyProduct.productType,
tags: shopifyProduct.tags,
available: shopifyProduct.availableForSale,
createdAt: shopifyProduct.createdAt,
updatedAt: shopifyProduct.updatedAt,
images: shopifyProduct.images.edges.map((edge: any) => ({
id: edge.node.id,
url: edge.node.url,
altText: edge.node.altText || '',
width: edge.node.width,
height: edge.node.height,
})),
featuredImage: shopifyProduct.featuredImage ? {
id: shopifyProduct.featuredImage.id,
url: shopifyProduct.featuredImage.url,
altText: shopifyProduct.featuredImage.altText || '',
width: shopifyProduct.featuredImage.width,
height: shopifyProduct.featuredImage.height,
} : undefined,
variants: shopifyProduct.variants.edges.map((edge: any) => ({
id: edge.node.id,
title: edge.node.title,
price: parseFloat(edge.node.price.amount),
compareAtPrice: edge.node.compareAtPrice?.amount
? parseFloat(edge.node.compareAtPrice.amount)
: undefined,
available: edge.node.availableForSale,
inventory: edge.node.quantityAvailable || 0,
selectedOptions: edge.node.selectedOptions,
image: edge.node.image ? {
id: edge.node.image.id,
url: edge.node.image.url,
altText: edge.node.image.altText || '',
width: edge.node.image.width,
height: edge.node.image.height,
} : undefined,
})),
options: shopifyProduct.options.map((option: any) => ({
id: option.id,
name: option.name,
values: option.values,
})),
metafields: shopifyProduct.metafields?.map((metafield: any) => ({
id: metafield.id || '',
key: metafield.key,
value: metafield.value,
type: metafield.type,
namespace: metafield.namespace,
})) || [],
};
}

function transformShopifyCollection(shopifyCollection: any): Collection {
return {
id: shopifyCollection.id,
title: shopifyCollection.title,
description: shopifyCollection.description || '',
handle: shopifyCollection.handle,
image: shopifyCollection.image ? {
id: shopifyCollection.image.id,
url: shopifyCollection.image.url,
altText: shopifyCollection.image.altText || '',
width: shopifyCollection.image.width,
height: shopifyCollection.image.height,
} : undefined,
products: [], // Will be populated separately if needed
};
}

// API functions
export async function fetchProductsFromShopify(options: {
first?: number;
sortKey?: string;
reverse?: boolean;
query?: string;
} = {}): Promise<Product[]> {
const {
first = 250,
sortKey = 'CREATED_AT',
reverse = true,
query = '',
} = options;

try {
const response = await client.request(PRODUCTS_QUERY, {
variables: { first, sortKey, reverse, query },
});

    if (response.data?.products?.edges) {
      return response.data.products.edges.map((edge: any) =>
        transformShopifyProduct(edge.node)
      );
    }

    return [];

} catch (error) {
console.error('Error fetching products from Shopify:', error);
throw error;
}
}

export async function fetchProductByHandle(handle: string): Promise<Product | null> {
try {
const response = await client.request(PRODUCT_BY_HANDLE_QUERY, {
variables: { handle },
});

    if (response.data?.productByHandle) {
      return transformShopifyProduct(response.data.productByHandle);
    }

    return null;

} catch (error) {
console.error('Error fetching product by handle:', error);
throw error;
}
}

export async function fetchCollections(): Promise<Collection[]> {
try {
const response = await client.request(COLLECTIONS_QUERY, {
variables: { first: 250 },
});

    if (response.data?.collections?.edges) {
      return response.data.collections.edges.map((edge: any) =>
        transformShopifyCollection(edge.node)
      );
    }

    return [];

} catch (error) {
console.error('Error fetching collections:', error);
throw error;
}
}

// Cart functions
export async function createCart() {
const CREATE_CART_MUTATION = `     mutation cartCreate($input: CartInput!) {
      cartCreate(input: $input) {
        cart {
          id
          lines(first: 100) {
            edges {
              node {
                id
                quantity
                merchandise {
                  ... on ProductVariant {
                    id
                    title
                    price {
                      amount
                      currencyCode
                    }
                    product {
                      id
                      title
                      handle
                      featuredImage {
                        url
                        altText
                      }
                    }
                  }
                }
              }
            }
          }
          estimatedCost {
            totalAmount {
              amount
              currencyCode
            }
            subtotalAmount {
              amount
              currencyCode
            }
            totalTaxAmount {
              amount
              currencyCode
            }
          }
        }
        userErrors {
          field
          message
        }
      }
    }
  `;

try {
const response = await client.request(CREATE_CART_MUTATION, {
variables: {
input: {},
},
});

    return response.data?.cartCreate?.cart;

} catch (error) {
console.error('Error creating cart:', error);
throw error;
}
}

export async function addToCart(cartId: string, variantId: string, quantity: number) {
const ADD_TO_CART_MUTATION = `     mutation cartLinesAdd($cartId: ID!, $lines: [CartLineInput!]!) {
      cartLinesAdd(cartId: $cartId, lines: $lines) {
        cart {
          id
          lines(first: 100) {
            edges {
              node {
                id
                quantity
                merchandise {
                  ... on ProductVariant {
                    id
                    title
                    price {
                      amount
                      currencyCode
                    }
                    product {
                      id
                      title
                      handle
                      featuredImage {
                        url
                        altText
                      }
                    }
                  }
                }
              }
            }
          }
          estimatedCost {
            totalAmount {
              amount
              currencyCode
            }
            subtotalAmount {
              amount
              currencyCode
            }
            totalTaxAmount {
              amount
              currencyCode
            }
          }
        }
        userErrors {
          field
          message
        }
      }
    }
  `;

try {
const response = await client.request(ADD_TO_CART_MUTATION, {
variables: {
cartId,
lines: [
{
merchandiseId: variantId,
quantity,
},
],
},
});

    return response.data?.cartLinesAdd?.cart;

} catch (error) {
console.error('Error adding to cart:', error);
throw error;
}
}

// Utility functions
export function filterProducts(products: Product[], filters: FilterOptions): Product[] {
let filteredProducts = [...products];

if (filters.priceRange) {
const [min, max] = filters.priceRange;
filteredProducts = filteredProducts.filter(
(product) => product.price >= min && product.price <= max,
);
}

if (filters.productTypes && filters.productTypes.length > 0) {
filteredProducts = filteredProducts.filter((product) =>
filters.productTypes!.includes(product.productType),
);
}

if (filters.tags && filters.tags.length > 0) {
filteredProducts = filteredProducts.filter((product) =>
filters.tags!.some((tag) => product.tags.includes(tag)),
);
}

if (filters.sortBy) {
filteredProducts.sort((a, b) => {
switch (filters.sortBy) {
case 'price-asc':
return a.price - b.price;
case 'price-desc':
return b.price - a.price;
case 'title-asc':
return a.title.localeCompare(b.title);
case 'title-desc':
return b.title.localeCompare(a.title);
case 'created-desc':
return new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime();
case 'created-asc':
return new Date(a.createdAt).getTime() - new Date(b.createdAt).getTime();
default:
return 0;
}
});
}

return filteredProducts;
}

export function formatPrice(price: number, currency: string = 'GBP'): string {
return new Intl.NumberFormat('en-GB', {
style: 'currency',
currency: currency,
}).format(price);
}
