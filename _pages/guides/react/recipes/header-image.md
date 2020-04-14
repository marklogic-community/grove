---
layout: inner
title: Grove React - Customize the Header Image
lead_text: ''
permalink: /guides/react/recipes/header-image
---

# Add an image to the top Header

You may prefer to have an image in the top left corner of the header navigation bar.  Grove's Navbar component was designed with this in mind and can be customized with a prop (or two if you'd like to style the image, too).

1. **Open `ui/src/components/Header.js`**.  A custom Header component that renders Grove's Navbar component, with links to "Search" and "Create" was provided as part of the starter Grove project.

2. **Copy the image you want to display in the Navbar to `ui/src/components`**. For this example, `Logo.png` was copied into this directory.

3. **Import the image**.  Because this project uses Webpack, you must import the image file directly into the component so that it will be included properly in the Webpack bundle.  In this example, importing a file provides a string value for the final path so that the `src` attribute of the <img/> DOM node can properly reference the file.

    ```javascript
      import myLogo from './Logo.png';
    ```

4. **Pass the image as a prop to Navbar**.  Add the `logo` prop, with `myLogo` as the value, to the Navbar child component.

    ```javascript
      <Navbar
        {...props}
        brandLink={brandLink}
        logo={mlLogo}
      >
    ```
    Go back to browser to inspect the result.  The default position may be suitable for your needs.  If it is not, continue to the next step.

5.  **(Optional) Style the image with an inline style**  The MLNavbar component also accepts a `logoStyle` prop.  This prop allows you to pass down an inline style object to assist with tasks such as sizing or positioning the image.  You may need to customize the styling to suit your needs and properly fit the image.  Note that a React inline style object uses camelCased properties rather than a CSS string.  The following is a simple example that restricts the size of the image and moves it slightly to align it better with the navigation options.

    ```javascript
      <Navbar
        {...props}
        brandLink={brandLink}
        logo={mlLogo}
        logoStyle={ {marginTop: '-5px', maxWidth: '100px', maxHeight: '45px'} }
      >
    ```

    Continue interating until you are happy with the styling.

And that's it.  You should now have your Grove application showing your custom logo.
