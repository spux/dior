# Dior (Data Island Only Read)

**Dior** is a simple library that extracts and manages structured data islands (`<script>` tags) embedded in HTML pages. Dior can retrieve and manipulate JSON or JSON-LD data embedded in web pages without needing additional network requests. It is designed to be lightweight and intuitive.

## Features
- **Extract structured data**: Retrieve data embedded in `<script>` tags of type `application/json` or `application/ld+json`.
- **Global access**: Access the extracted data via the `di` object globally on your page.
- **Query string parsing**: Automatically parse query string parameters into a global `qs` object.
- **Automatic updates**: Update data islands directly in the DOM by modifying the `di` object.

## Installation

You can use Dior by adding the script directly to your HTML page. No build step is needed.

### Example HTML Setup

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Dior Example</title>
</head>
<body>
  <!-- Data Island -->
  <script id="app-data" type="application/json">
  {
    "user": {
      "name": "Alice",
      "role": "admin"
    },
    "theme": "dark"
  }
  </script>

  <!-- Include Dior Script -->
  <script>
    ;(() => {
      globalThis.qs = Object.fromEntries(
        new URLSearchParams(document.location.search)
      )

      globalThis.di = new Proxy(
        Array.from(
          document.querySelectorAll(
            'script[type="application/ld+json"], script[type="application/json"]'
          )
        )
          .map(island => [island.id, JSON.parse(island.text || JSON.stringify(''))])
          .reduce((obj, item) => {
            obj[item[0]] = item[1]
            return obj
          }, {}),
        {
          set: (obj, prop, value) => {
            obj[prop] = value
            var island = document.getElementById(prop)
            island.text = JSON.stringify(value, null, 2)
            return true
          }
        }
      )
    })()
  </script>

  <script>
    // Use Dior to access and update the structured data
    console.log('Data Island:', di['app-data']);
    di['app-data'].theme = 'light';  // Update the theme and DOM
  </script>
</body>
</html>
```

## How It Works

1. **Global Query String (`qs`)**: The `qs` object automatically parses the query parameters from the URL, making them accessible globally.
   
   Example:
   ```javascript
   // URL: example.com?page=home
   console.log(qs.page);  // Output: 'home'
   ```

2. **Global Data Island Object (`di`)**: Dior automatically retrieves all `<script>` elements with `type="application/json"` or `type="application/ld+json"`, parses their contents, and stores them in the global `di` object, using the script's `id` as the key.

   Example:
   ```javascript
   console.log(di['app-data']);  // Output: { user: { name: 'Alice', role: 'admin' }, theme: 'dark' }
   ```

3. **Automatic DOM Updates**: You can modify the contents of the `di` object, and Dior will automatically update the corresponding `<script>` tag in the DOM.

   Example:
   ```javascript
   di['app-data'].theme = 'light';
   // This automatically updates the `app-data` script content in the DOM.
   ```

## Example Usage

### Access Data
You can easily access any data island by its `id` using the `di` object.

```javascript
// Accessing data from the app-data island
const appData = di['app-data'];
console.log(appData.user.name);  // Output: 'Alice'
```

### Update Data
You can also update the data within the script tag, and the DOM will automatically reflect the changes.

```javascript
// Changing the theme to light mode
di['app-data'].theme = 'light';
```

This will automatically update the corresponding `<script>` tag in the DOM:

```html
<script id="app-data" type="application/json">
{
  "user": {
    "name": "Alice",
    "role": "admin"
  },
  "theme": "light"
}
</script>
```

### Access Query Parameters

Dior also parses the query string and makes it accessible via the `qs` object.

```javascript
// URL: example.com?user=alice&theme=dark
console.log(qs.user);  // Output: 'alice'
console.log(qs.theme);  // Output: 'dark'
```

## Limitations

- Dior only works with `<script>` tags containing `application/json` or `application/ld+json` types.
- It’s meant to handle read and updates of structured data but is not designed for complex data binding or event handling.

## Conclusion

Dior provides a lightweight solution for managing structured data islands in web pages, giving you the ability to access and update data directly from the DOM with minimal overhead. It offers simplicity for small-scale applications where data is embedded directly in HTML, reducing the need for additional requests to initialize the app.

Feel free to drop Dior into your web projects for fast, reliable data management on the client side!
